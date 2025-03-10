# Docker 아키텍처 및 스토리지 관리

## Docker 아키텍처

Docker는 레이어드(Layered) 아키텍처를 기반으로 동작하며, 이를 통해 이미지 빌드를 효율적으로 수행하고 빠른 배포 및 디스크 공간 절약이 가능하다. 각 레이어는 이전 레이어에서 변경된 내용만 저장하여 중복을 최소화하며, 이러한 구조 덕분에 여러 컨테이너가 동일한 베이스 이미지를 공유할 수 있어 자원 활용이 최적화된다.

## Docker 볼륨과 영구 저장소

Docker 컨테이너는 기본적으로 임시 저장소를 사용하기 때문에 컨테이너가 종료되면 내부의 데이터도 함께 삭제된다. 이를 방지하기 위해 Docker는 볼륨(Volume)과 바인드 마운트(Bind Mount)를 제공하여 데이터를 영구적으로 저장할 수 있도록 한다. 

볼륨은 Docker가 직접 관리하는 저장소로, `/var/lib/docker/volumes/` 디렉터리에 저장되며, 컨테이너 간 공유가 가능하다. 

예를 들어, ```
```
docker volume create data_volume
```
=>  새로운 볼륨이 생성

이를 컨테이너에 연결하려면 
```
docker run -v data_volume:/var/lib/mysql mysql
```

바인드 마운트: 특정 호스트 디렉터리를 컨테이너 내부에 직접 마운트하는 방식
```
docker run \
--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

볼륨 마운트: Docker가 데이터를 자동으로 관리하기 
=> 보안성과 이동성이 뛰어남

바인드 마운트:  컨테이너에서 호스트의 특정 디렉터리에 직접 접근해야 할 때 유용

볼륨을 활용하면 컨테이너 삭제 후에도 데이터가 유지되므로, 데이터베이스와 같은 지속적인 저장소가 필요한 서비스에서 많이 사용

## Docker 스토리지 드라이버

Docker는 여러 개의 레이어를 효율적으로 관리하기 위해 다양한 스토리지 드라이버를 사용한다. 
운영체제에 따라 기본적으로 사용되는 스토리지 드라이버가 다르다, 
- 대표적인 드라이버로는 AUFS, ZFS, OverlayFS, Overlay2 등이 있다. 
- Ubuntu: OverlayFS, AUFS가 주로 사용
- CentOS 및 RHEL 계열: Overlay2가 기본으로 설정.
 
스토리지 드라이버는 컨테이너 파일 시스템의 성능과 동작 방식에 영향을 미치므로, 운영 환경에 맞는 드라이버를 선택하는 것이 중요하다.

## Docker 볼륨 드라이버와 외부 스토리지

Docker는 기본적으로 로컬 볼륨 드라이버를 사용하지만, 외부 스토리지 솔루션과 연동할 수 있도록 다양한 볼륨 드라이버 플러그인을 제공
-  AWS EBS, Google Cloud Persistent Disk, Azure File Storage 등의 클라우드 스토리지를 사용
 ```
docker volume create --driver rexray --opt size=50G my_aws_volume
```
- AWS EBS에 50GB 크기의 볼륨을 생성
- 이를 활용하면 데이터가 클라우드에 안전하게 저장되며, 여러 컨테이너에서 동일한 데이터를 공유할 수 있다.

## 컨테이너 스토리지 인터페이스(Container Storage Interface, CSI)

CSI는 컨테이너 오케스트레이션 시스템에서 다양한 스토리지 솔루션을 통합할 수 있도록 설계된 표준 인터페이스

- CSI를 사용하면 벤더에 관계없이 다양한 스토리지를 컨테이너 환경에서 유연하게 사용
- 개발자가 자체적인 스토리지 드라이버를 구현. 

이를 통해 AWS, GCP, Ceph, NFS, iSCSI 등의 다양한 스토리지 백엔드와 연결할 수 있으며, Kubernetes와 같은 오케스트레이션 환경에서 자동 볼륨 프로비저닝 및 관리가 가능

## Docker 볼륨을 활용한 데이터 영구 보존

```sh
# 볼륨 생성
docker volume create my_data

# MySQL 컨테이너 실행 (볼륨 연결)
docker run -d --name mysql-db -v my_data:/var/lib/mysql mysql

# 컨테이너 삭제 후에도 데이터 유지됨
docker stop mysql-db
docker rm mysql-db

# 동일한 볼륨으로 새 컨테이너 실행
docker run -d --name mysql-db2 -v my_data:/var/lib/mysql mysql
```

이제 MySQL 컨테이너를 삭제해도 my_data 볼륨에 저장된 데이터는 유지되며, 다시 컨테이너를 실행하면 기존 데이터를 그대로 사용할 수 있다.

## Volumes

```
volumes:
- name: data-volume
hostPath:
path: /data
type: Directory
```

![](https://i.imgur.com/f4M8NGZ.png)


aws EBS라면
```
volumes:
- name: data-volume
hostPath:
path: /data
type: Direct
```

> 기본적으로 Kubernetes에서 각 사용자가 매번 볼륨을 개별적으로 생성하여 Pod에 할당해야 하지만, 이러한 방식은 비효율적
개별적으로 관리하는 것보다 **중앙에서 볼륨을 관리하고 필요한 Pod에서 이를 요청하여 사용하는 것이 더 효율적** => **Persistent Volume(PV)** 

## Persistent Volume (PV)란?
Persistent Volume은 **클러스터 전체에서 관리되는 스토리지 볼륨**으로, 클러스터 관리자가 미리 설정하고, 사용자는 이를 필요할 때 요청하여 사용할 수 있다. 
이는 쿠버네티스에서 제공하는 스토리지 리소스로, 사용자가 직접 볼륨을 생성하지 않고도 기존에 정의된 스토리지를 활용할 수 있도록 한다.

## PV의 동작 방식
1. **클러스터 관리자가 Persistent Volume(PV)을 미리 설정**하여 사용 가능한 스토리지 풀을 정의
2. **사용자가 Persistent Volume Claim(PVC)를 통해 스토리지를 요청**하면, PVC의 요구 사항과 일치하는 PV가 자동으로 할당
3. **Pod에서 PVC를 사용하여 영구적인 스토리지에 접근**할 수 있으며, Pod가 삭제되더라도 데이터는 유지.

## PV의 주요 속성
PV는 다양한 접근 모드를 제공하며, 클러스터의 스토리지 정책에 따라 다른 유형으로 구성할 수 있다.
### Access Modes (액세스 모드)
- **ReadWriteOnce(RWO)**: 하나의 노드에서만 읽기/쓰기 가능.
- **ReadOnlyMany(ROX)**: 여러 노드에서 읽기만 가능.
- **ReadWriteMany(RWX)**: 여러 노드에서 동시에 읽기/쓰기 가능.

## Persistent Volume 정의 예제

아래는 AWS EBS 볼륨을 사용하는 Persistent Volume을 정의하는 예제이다.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```
이 설정을 통해 1Gi 크기의 AWS EBS 볼륨을 생성하고, 이를 ReadWriteOnce 모드로 사용할 수 있도록 한다.
### Persistent Volume Claim
![](https://i.imgur.com/ZhVbrhu.png)
# Persistent Volume Claim (PVC)

쿠버네티스에서 **Persistent Volume Claim(PVC)** 는 사용자가 스토리지를 요청할 때 사용하는 리소스다.  

PV(영구 볼륨)는 클러스터 관리자가 미리 생성하지만, 개별 사용자나 애플리케이션은 PVC를 통해 필요할 때 PV를 요청하고 할당받는다.  
PVC가 생성되면, 쿠버네티스는 사용자의 요청에 맞는 PV를 자동으로 찾아서 바인딩을 수행한다.

---
### PVC의 동작 방식
- PVC와 PV는 기본적으로 **1:1 매칭**되며, 한번 바인딩되면 변경되지 않는다.
- 바인딩 과정에서 쿠버네티스는 다음과 같은 조건을 만족하는 PV를 찾는다.
  - 요청한 **스토리지 용량**(capacity)
  - **Access Modes**(읽기/쓰기 모드)
  - **Volume Mode**(파일시스템 기반 또는 블록 디바이스)
  - **Storage Class**(스토리지 유형)
  - **Selector**(라벨 기반 매칭)

PVC는 라벨과 셀렉터(selector)를 사용하여 특정 PV를 요청할 수도 있다.  
만약 요청한 조건에 맞는 PV가 없으면 PVC는 **Pending** 상태로 남아 있으며, 새로운 PV가 생성될 때까지 기다린다.  
조건에 완벽히 맞는 PV가 없다면 **보다 큰 용량의 PV와 매칭**될 수도 있다.

500Mi` 크기의 `ReadWriteOnce` 액세스 모드를 요청하는 PVC의 정의
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

PVC가 생성되면, 쿠버네티스는 해당 PVC의 요청에 맞는 PV를 자동으로 찾아 바인딩한다.

---
#### Persistent Volume Reclaim Policy (재클레임 정책)

PVC가 삭제되었을 때, PV의 처리 방법을 결정하는 ReclaimPolicy가 존재한다.
기본적으로 Retain이 설정되어 있으며, 경우에 따라 Delete로 변경할 수 있다.

ReclaimPolicy	설명
Retain (기본값)	PVC가 삭제되더라도 PV를 유지하며, 수동으로 정리해야 한다.
Delete	PVC가 삭제되면 PV도 함께 삭제된다.

> PVC를 삭제하면 PV의 claimRef가 여전히 삭제된 PVC를 가리키고 있어 자동으로 재사용되지 않는다.
이 경우, PV를 재사용하려면 수동으로 claimRef를 제거한 후 다시 바인딩해야 한다.
반면, Delete 정책이 설정되어 있다면 PVC 삭제 시 PV도 자동으로 삭제된다.

---
# Storage Class

기본적으로 Kubernetes에서 PVC를 사용하려면 **미리 PV를 수동으로 생성**해야 한다.  
즉, PVC를 만들기 전에 **수동으로 PV 파일을 생성하고, 동일한 이름으로 매칭해야 한다.**  
이러한 방식을 **Static Provisioning(정적 프로비저닝)** 

하지만, 매번 수동으로 PV를 생성하는 것은 번거롭기 때문에 **Storage Class**를 사용하면 PVC 요청 시 **자동으로 볼륨을 생성**할 수 있다.  
이러한 방식은 **Dynamic Provisioning(동적 프로비저닝)** 이라고 하며, Storage Class는 이를 가능하게 해준다.

Storage Class는
- PVC 요청이 발생하면 **자동으로 PV를 생성하고, Pod에 연결**해준다.
- Storage Class를 지정하면 Kubernetes가 적절한 스토리지 백엔드를 활용하여 볼륨을 프로비저닝한다.
- 다양한 서비스 등급(Silver, Gold, Platinum)을 제공하여 **차별화된 스토리지 성능을 설정**할 수 있다.

아래는 Google Cloud Storage를 활용하여 **자동으로 볼륨을 생성하는 Storage Class**의 정의다.
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

이제 PVC에서 storageClassName을 설정하면 자동으로 해당 Storage Class를 사용하여 PV가 생성된다.

PVC에서 특정 Storage Class를 사용하려면, storageClassName을 지정하면 된다.
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: google-storage
```

이 PVC가 생성되면, google-storage Storage Class에 의해 자동으로 PV가 생성되고 바인딩된다.
즉, 이제 더 이상 수동으로 PV를 만들 필요가 없다.

Storage Class의 추가 옵션

provisioner
- 특정 스토리지 백엔드와 연동하여 자동으로 PV를 생성하는 역할을 한다.
- 예시:
	- kubernetes.io/gce-pd → Google Cloud Storage
	- kubernetes.io/aws-ebs → AWS EBS
	- kubernetes.io/azure-disk → Azure Disk

waitForFirstConsumer
	•	PVC가 생성되더라도 실제 Pod에서 사용되기 전까지 PV를 바인딩하지 않음.
	•	즉, 사용되지 않는 볼륨을 미리 할당하지 않고, Pod가 스케줄링될 때만 볼륨을 프로비저닝한다.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-storage
provisioner: kubernetes.io/gce-pd
volumeBindingMode: WaitForFirstConsumer
```

이렇게 설정하면 PVC가 바로 PV와 바인딩되지 않고,
Pod가 실제로 사용해야 할 때만 바인딩된다.

### 정리
Storage Class를 활용한 효율적인 스토리지 관리
	1.	Storage Class를 정의하여 동적 프로비저닝을 활성화한다.
	2.	PVC에서 storageClassName을 지정하여 자동으로 적절한 PV가 생성되도록 설정한다.
	3.	waitForFirstConsumer 옵션을 사용하면 불필요한 볼륨 할당을 방지하고 자원 낭비를 줄일 수 있다.



