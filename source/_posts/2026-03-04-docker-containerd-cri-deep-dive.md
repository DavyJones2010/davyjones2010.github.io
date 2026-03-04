---
title: Docker、containerd、CRI 深度剖析
date: 2026-03-04 21:42:05
tags: [k8s, container, docker, containerd, cri, deep-dive]
---

# docker-containerd-cri-deep-dive

# Overview

![Untitled](/_assets/2026-03-04-docker-containerd-cri-deep-dive/Untitled.png)

![Untitled](/_assets/2026-03-04-docker-containerd-cri-deep-dive/Untitled-1.png)

个人总结下. 

- containerd本质就是 runc + image 下载/解压
- docker本质就是 containerd + image 构建

> Docker was focused on end-user and developer use cases, 
whereas containerd is focused on operational use cases, such as running containers on servers. 
Tasks such as building container images are left to other tools.
> 

- 而根据CRI的接口定义, 也不包含镜像的构建:
    - containerd就是CRI的一个实现.
    - cri-o 也是CRI的一个实现.
    - 但目前主流的都是 containerd

# API

## CRI API

CRI的API定义: [cri-api/pkg/apis/runtime/v1/api.proto· kubernetes/cri-api](https://github.com/kubernetes/cri-api/blob/c75ef5b/pkg/apis/runtime/v1/api.proto)

### PodSandbox API

```go

service RuntimeService {
    rpc RunPodSandbox(RunPodSandboxRequest) returns (RunPodSandboxResponse) {}
    rpc StopPodSandbox(StopPodSandboxRequest) returns (StopPodSandboxResponse) {}
    rpc RemovePodSandbox(RemovePodSandboxRequest) returns (RemovePodSandboxResponse) {}
    rpc PodSandboxStatus(PodSandboxStatusRequest) returns (PodSandboxStatusResponse) {}
    rpc ListPodSandbox(ListPodSandboxRequest) returns (ListPodSandboxResponse) {}
}
```

sandboxConfig里的重要信息: 

```go

type PodSandboxConfig struct {
	Metadata *PodSandboxMetadata `protobuf:"bytes,1,opt,name=metadata,proto3" json:"metadata,omitempty"`
  Hostname string `protobuf:"bytes,2,opt,name=hostname,proto3" json:"hostname,omitempty"`
  LogDirectory string `protobuf:"bytes,3,opt,name=log_directory,json=logDirectory,proto3" json:"log_directory,omitempty"`
	DnsConfig *DNSConfig `protobuf:"bytes,4,opt,name=dns_config,json=dnsConfig,proto3" json:"dns_config,omitempty"`
	PortMappings []*PortMapping `protobuf:"bytes,5,rep,name=port_mappings,json=portMappings,proto3" json:"port_mappings,omitempty"`
  Linux *LinuxPodSandboxConfig `protobuf:"bytes,8,opt,name=linux,proto3" json:"linux,omitempty"`
}

type PodSandboxMetadata struct {
	Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
	Uid string `protobuf:"bytes,2,opt,name=uid,proto3" json:"uid,omitempty"`
	Namespace string `protobuf:"bytes,3,opt,name=namespace,proto3" json:"namespace,omitempty"`
}

type LinuxPodSandboxConfig struct {
	CgroupParent string `protobuf:"bytes,1,opt,name=cgroup_parent,json=cgroupParent,proto3" json:"cgroup_parent,omitempty"`
	Overhead *LinuxContainerResources `protobuf:"bytes,4,opt,name=overhead,proto3" json:"overhead,omitempty"`
	Resources            *LinuxContainerResources `protobuf:"bytes,5,opt,name=resources,proto3" json:"resources,omitempty"`
}

type LinuxContainerResources struct {
	CpuPeriod int64 `protobuf:"varint,1,opt,name=cpu_period,json=cpuPeriod,proto3" json:"cpu_period,omitempty"`
	CpuQuota int64 `protobuf:"varint,2,opt,name=cpu_quota,json=cpuQuota,proto3" json:"cpu_quota,omitempty"`
	CpuShares int64 `protobuf:"varint,3,opt,name=cpu_shares,json=cpuShares,proto3" json:"cpu_shares,omitempty"`
	MemoryLimitInBytes int64 `protobuf:"varint,4,opt,name=memory_limit_in_bytes,json=memoryLimitInBytes,proto3" json:"memory_limit_in_bytes,omitempty"`
	OomScoreAdj int64 `protobuf:"varint,5,opt,name=oom_score_adj,json=oomScoreAdj,proto3" json:"oom_score_adj,omitempty"`
	CpusetCpus string `protobuf:"bytes,6,opt,name=cpuset_cpus,json=cpusetCpus,proto3" json:"cpuset_cpus,omitempty"`
	CpusetMems string `protobuf:"bytes,7,opt,name=cpuset_mems,json=cpusetMems,proto3" json:"cpuset_mems,omitempty"`
  ...
}

message RunPodSandboxResponse {
    // ID of the PodSandbox to run.
    string pod_sandbox_id = 1;
}
```

### Container API

```go
service RuntimeService {
		// CreateContainer creates a new container in specified PodSandbox
    rpc CreateContainer(CreateContainerRequest) returns (CreateContainerResponse) {}
    // StartContainer starts the container.
    rpc StartContainer(StartContainerRequest) returns (StartContainerResponse) {}
    // StopContainer stops a running container with a grace period (i.e., timeout).
    // This call is idempotent, and must not return an error if the container has
    // already been stopped.
    // The runtime must forcibly kill the container after the grace period is
    // reached.
    rpc StopContainer(StopContainerRequest) returns (StopContainerResponse) {}
    // RemoveContainer removes the container. If the container is running, the
    // container must be forcibly removed.
    // This call is idempotent, and must not return an error if the container has
    // already been removed.
    rpc RemoveContainer(RemoveContainerRequest) returns (RemoveContainerResponse) {}
    // ListContainers lists all containers by filters.
    rpc ListContainers(ListContainersRequest) returns (ListContainersResponse) {}
    // ContainerStatus returns status of the container. If the container is not
    // present, returns an error.
    rpc ContainerStatus(ContainerStatusRequest) returns (ContainerStatusResponse) {}
    // UpdateContainerResources updates ContainerConfig of the container synchronously.
    // If runtime fails to transactionally update the requested resources, an error is returned.
    rpc UpdateContainerResources(UpdateContainerResourcesRequest) returns (UpdateContainerResourcesResponse) {}
    // ReopenContainerLog asks runtime to reopen the stdout/stderr log file
    // for the container. This is often called after the log file has been
    // rotated. If the container is not running, container runtime can choose
    // to either create a new log file and return nil, or return an error.
    // Once it returns error, new container log file MUST NOT be created.
    rpc ReopenContainerLog(ReopenContainerLogRequest) returns (ReopenContainerLogResponse) {}
}
```

### Image API

```go
// ImageService defines the public APIs for managing images.
service ImageService {
    // ListImages lists existing images.
    rpc ListImages(ListImagesRequest) returns (ListImagesResponse) {}
    // ImageStatus returns the status of the image. If the image is not
    // present, returns a response with ImageStatusResponse.Image set to
    // nil.
    rpc ImageStatus(ImageStatusRequest) returns (ImageStatusResponse) {}
    // PullImage pulls an image with authentication config.
    rpc PullImage(PullImageRequest) returns (PullImageResponse) {}
    // RemoveImage removes the image.
    // This call is idempotent, and must not return an error if the image has
    // already been removed.
    rpc RemoveImage(RemoveImageRequest) returns (RemoveImageResponse) {}
    // ImageFSInfo returns information of the filesystem that is used to store images.
    rpc ImageFsInfo(ImageFsInfoRequest) returns (ImageFsInfoResponse) {}
}
```

### CRI-API如何编排? 在哪里编排?

- 上边几个API如何编排? 样例如下:

```go
// 第一步: 先拉取镜像
ImageService.PullImage({image: "image1"})
ImageService.PullImage({image: "image2"})

// 第二步: 创建sandbox/pod
podID = RuntimeService.RunPodSandbox({name: "mypod"})

// 第三步: 在sandbox里创建container
id1 = RuntimeService.CreateContainer({
  pod: podID,
  name: "container1",
  image: "image1",
})
id2 = RuntimeService.CreateContainer({
  pod: podID,
  name: "container2",
  image: "image2",
})

// 第四步: 启动容器
RuntimeService.StartContainer({id: id1})
RuntimeService.StartContainer({id: id2})
```

- 实际是在kubelet里进行编排, 代码如下:

<aside>
💡 - pkg/kubelet/kuberuntime/kuberuntime_manager.go:1054 `SyncPod` 通过调用CRI实现, 创建sandbox
- pkg/kubelet/kuberuntime/kuberuntime_container.go:179 `startContainer` 通过调用CRI实现, 来在sandbox里拉取镜像/创建/启动容器

</aside>

- 当然自己也可以使用 crictl 来手动编排, 创建个自己的容器, 参见: [cri-containerd-cli](cri-containerd-runc-cli%20b82e027079204eac9df7e44aeca1b0ec.md)

## Containerd API

即containerd提供的一系列API, 用于底层runtime来实现. containerd会通过调用这些API, 来完成一系列操作. 

// TODO: 感觉与CRI API基本一致, 为啥还要单独定义呢? 

### Sandbox API

设计参见: 

```go

// sandboxer必须实现 SandboxServer, 以grpc形式提供出来. 
type SandboxServer interface {
	// CreateSandbox will be called right after sandbox shim instance launched.
	// It is a good place to initialize sandbox environment.
	CreateSandbox(context.Context, *CreateSandboxRequest) (*CreateSandboxResponse, error)
	// StartSandbox will start a previously created sandbox.
	StartSandbox(context.Context, *StartSandboxRequest) (*StartSandboxResponse, error)
	// Platform queries the platform the sandbox is going to run containers on.
	// containerd will use this to generate a proper OCI spec.
	Platform(context.Context, *PlatformRequest) (*PlatformResponse, error)
	// StopSandbox will stop existing sandbox instance
	StopSandbox(context.Context, *StopSandboxRequest) (*StopSandboxResponse, error)
	// WaitSandbox blocks until sandbox exits.
	WaitSandbox(context.Context, *WaitSandboxRequest) (*WaitSandboxResponse, error)
	// SandboxStatus will return current status of the running sandbox instance
	SandboxStatus(context.Context, *SandboxStatusRequest) (*SandboxStatusResponse, error)
	// PingSandbox is a lightweight API call to check whether sandbox alive.
	PingSandbox(context.Context, *PingRequest) (*PingResponse, error)
	// ShutdownSandbox must shutdown shim instance.
	ShutdownSandbox(context.Context, *ShutdownSandboxRequest) (*ShutdownSandboxResponse, error)
	// SandboxMetrics retrieves metrics about a sandbox instance.
	SandboxMetrics(context.Context, *SandboxMetricsRequest) (*SandboxMetricsResponse, error)
	mustEmbedUnimplementedSandboxServer()
}

```

### Containers API

```go
service Containers {
	rpc Get(GetContainerRequest) returns (GetContainerResponse);
	rpc List(ListContainersRequest) returns (ListContainersResponse);
	rpc ListStream(ListContainersRequest) returns (stream ListContainerMessage);
	rpc Create(CreateContainerRequest) returns (CreateContainerResponse);
	rpc Update(UpdateContainerRequest) returns (UpdateContainerResponse);
	rpc Delete(DeleteContainerRequest) returns (google.protobuf.Empty);
}
message Container {
	// ID is the user-specified identifier.
	//
	// This field may not be updated.
	string id = 1;

	// Labels provides an area to include arbitrary data on containers.
	//
	// The combined size of a key/value pair cannot exceed 4096 bytes.
	//
	// Note that to add a new value to this field, read the existing set and
	// include the entire result in the update call.
	map<string, string> labels  = 2;

	// Image contains the reference of the image used to build the
	// specification and snapshots for running this container.
	//
	// If this field is updated, the spec and rootfs needed to updated, as well.
	string image = 3;

	message Runtime {
		// Name is the name of the runtime.
		string name = 1;
		// Options specify additional runtime initialization options.
		google.protobuf.Any options = 2;
	}
	// Runtime specifies which runtime to use for executing this container.
	Runtime runtime = 4;

	// Spec to be used when creating the container. This is runtime specific.
	google.protobuf.Any spec = 5;

	// Snapshotter specifies the snapshotter name used for rootfs
	string snapshotter = 6;

	// SnapshotKey specifies the snapshot key to use for the container's root
	// filesystem. When starting a task from this container, a caller should
	// look up the mounts from the snapshot service and include those on the
	// task create request.
	//
	// Snapshots referenced in this field will not be garbage collected.
	//
	// This field is set to empty when the rootfs is not a snapshot.
	//
	// This field may be updated.
	string snapshot_key = 7;

	// CreatedAt is the time the container was first created.
	google.protobuf.Timestamp created_at = 8;

	// UpdatedAt is the last time the container was mutated.
	google.protobuf.Timestamp updated_at = 9;

	// Extensions allow clients to provide zero or more blobs that are directly
	// associated with the container. One may provide protobuf, json, or other
	// encoding formats. The primary use of this is to further decorate the
	// container object with fields that may be specific to a client integration.
	//
	// The key portion of this map should identify a "name" for the extension
	// that should be unique against other extensions. When updating extension
	// data, one should only update the specified extension using field paths
	// to select a specific map key.
	map<string, google.protobuf.Any> extensions = 10;

	// Sandbox ID this container belongs to.
	string sandbox = 11;
}
```

### Image API

```go
service Images {
	// Get returns an image by name.
	rpc Get(GetImageRequest) returns (GetImageResponse);

	// List returns a list of all images known to containerd.
	rpc List(ListImagesRequest) returns (ListImagesResponse);

	// Create an image record in the metadata store.
	//
	// The name of the image must be unique.
	rpc Create(CreateImageRequest) returns (CreateImageResponse);

	// Update assigns the name to a given target image based on the provided
	// image.
	rpc Update(UpdateImageRequest) returns (UpdateImageResponse);

	// Delete deletes the image by name.
	rpc Delete(DeleteImageRequest) returns (google.protobuf.Empty);
}
message Image {
	// Name provides a unique name for the image.
	//
	// Containerd treats this as the primary identifier.
	string name = 1;

	// Labels provides free form labels for the image. These are runtime only
	// and do not get inherited into the package image in any way.
	//
	// Labels may be updated using the field mask.
	// The combined size of a key/value pair cannot exceed 4096 bytes.
	map<string, string> labels = 2;
}
```

### Snapshot API

```go
// Snapshot service manages snapshots
service Snapshots {
	rpc Prepare(PrepareSnapshotRequest) returns (PrepareSnapshotResponse);
	rpc View(ViewSnapshotRequest) returns (ViewSnapshotResponse);
	rpc Mounts(MountsRequest) returns (MountsResponse);
	rpc Commit(CommitSnapshotRequest) returns (google.protobuf.Empty);
	rpc Remove(RemoveSnapshotRequest) returns (google.protobuf.Empty);
	rpc Stat(StatSnapshotRequest) returns (StatSnapshotResponse);
	rpc Update(UpdateSnapshotRequest) returns (UpdateSnapshotResponse);
	rpc List(ListSnapshotsRequest) returns (stream ListSnapshotsResponse);
	rpc Usage(UsageRequest) returns (UsageResponse);
	rpc Cleanup(CleanupRequest) returns (google.protobuf.Empty);
}

message PrepareSnapshotRequest {
	string snapshotter = 1;
	string key = 2;
	string parent = 3;

	// Labels are arbitrary data on snapshots.
	//
	// The combined size of a key/value pair cannot exceed 4096 bytes.
	map<string, string> labels  = 4;
}
```

### Task API

```go
service Tasks {
	// Create a task.
	rpc Create(CreateTaskRequest) returns (CreateTaskResponse);

	// Start a process.
	rpc Start(StartRequest) returns (StartResponse);

	// Delete a task and on disk state.
	rpc Delete(DeleteTaskRequest) returns (DeleteResponse);

	rpc DeleteProcess(DeleteProcessRequest) returns (DeleteResponse);

	rpc Get(GetRequest) returns (GetResponse);

	rpc List(ListTasksRequest) returns (ListTasksResponse);

	// Kill a task or process.
	rpc Kill(KillRequest) returns (google.protobuf.Empty);

	rpc Exec(ExecProcessRequest) returns (google.protobuf.Empty);

	rpc ResizePty(ResizePtyRequest) returns (google.protobuf.Empty);

	rpc CloseIO(CloseIORequest) returns (google.protobuf.Empty);

	rpc Pause(PauseTaskRequest) returns (google.protobuf.Empty);

	rpc Resume(ResumeTaskRequest) returns (google.protobuf.Empty);

	rpc ListPids(ListPidsRequest) returns (ListPidsResponse);

	rpc Checkpoint(CheckpointTaskRequest) returns (CheckpointTaskResponse);

	rpc Update(UpdateTaskRequest) returns (google.protobuf.Empty);

	rpc Metrics(MetricsRequest) returns (MetricsResponse);

	rpc Wait(WaitRequest) returns (WaitResponse);
}

message CreateTaskRequest {
	string container_id = 1;

	// RootFS provides the pre-chroot mounts to perform in the shim before
	// executing the container task.
	//
	// These are for mounts that cannot be performed in the user namespace.
	// Typically, these mounts should be resolved from snapshots specified on
	// the container object.
	repeated containerd.types.Mount rootfs = 3;

	string stdin = 4;
	string stdout = 5;
	string stderr = 6;
	bool terminal = 7;

	containerd.types.Descriptor checkpoint = 8;

	google.protobuf.Any options = 9;

	string runtime_path = 10;
}
```

## OCI API

OCI runtime spec: [runtime-spec/runtime.md at main · opencontainers/runtime-spec](https://github.com/opencontainers/runtime-spec/blob/main/runtime.md)

# 总体交互流程

![Untitled](/_assets/2026-03-04-docker-containerd-cri-deep-dive/Untitled-2.png)

# 详细

containerd如何实现CRI接口? 

→ 客户端可以这样调用, 初始化client, 然后底层是通过grpc调用 RunPodSandbox 请求

```go
// RunPodSandbox creates and starts a pod-level sandbox. Runtimes should ensure
// the sandbox is in ready state.
func (r *RuntimeService) RunPodSandbox(config *runtimeapi.PodSandboxConfig, runtimeHandler string, opts ...grpc.CallOption) (string, error) {
	// Use 2 times longer timeout for sandbox operation (4 mins by default)
	// TODO: Make the pod sandbox timeout configurable.
	timeout := r.timeout * 2

	klog.V(10).Infof("[RuntimeService] RunPodSandbox (config=%v, runtimeHandler=%v, timeout=%v)", config, runtimeHandler, timeout)

	ctx, cancel := getContextWithTimeout(timeout)
	defer cancel()

	resp, err := r.runtimeClient.RunPodSandbox(ctx, &runtimeapi.RunPodSandboxRequest{
		Config:         config,
		RuntimeHandler: runtimeHandler,
	}, opts...)
	if err != nil {
		klog.Errorf("RunPodSandbox from runtime service failed: %v", err)
		return "", err
	}

	if resp.PodSandboxId == "" {
		errorMessage := fmt.Sprintf("PodSandboxId is not set for sandbox %q", config.GetMetadata())
		klog.Errorf("RunPodSandbox failed: %s", errorMessage)
		return "", errors.New(errorMessage)
	}

	klog.V(10).Infof("[RuntimeService] RunPodSandbox Response (PodSandboxId=%v)", resp.PodSandboxId)

	return resp.PodSandboxId, nil
}
```

→ 服务端这样实现, 统一入口是: `instrumented_service.go` 

containerd如何实现CRI 

```go

func (in *instrumentedService) RunPodSandbox(ctx context.Context, r *runtime.RunPodSandboxRequest) (res *runtime.RunPodSandboxResponse, err error) {
	if err := in.checkInitialized(); err != nil {
		return nil, err
	}
	log.G(ctx).Infof("RunPodSandbox for %+v", r.GetConfig().GetMetadata())
	defer func() {
		if err != nil {
			log.G(ctx).WithError(err).Errorf("RunPodSandbox for %+v failed, error", r.GetConfig().GetMetadata())
		} else {
			log.G(ctx).Infof("RunPodSandbox for %+v returns sandbox id %q", r.GetConfig().GetMetadata(), res.GetPodSandboxId())
		}
	}()
	res, err = in.c.RunPodSandbox(ctrdutil.WithNamespace(ctx), r)
	return res, errdefs.ToGRPC(err)
}
```

# Refs

- [The differences between Docker, containerd, CRI-O and runc - Tutorial Works](https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/)
- [Container Runtimes Part 4: Kubernetes Container Runtimes & CRI](https://www.ianlewis.org/en/container-runtimes-part-4-kubernetes-container-run)