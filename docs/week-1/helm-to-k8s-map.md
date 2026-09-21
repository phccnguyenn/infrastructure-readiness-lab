
1. Application runtime contract
   Image, port, env, health, dependencies
   ↓
2. Đọc chart interface
   values.yaml + templates
   ↓
3. Điền application baseline
   config/app/go-click-tracker.yaml
   ↓
4. Điền local/release override
   image tag, replicas, local storage...
   ↓
5. Helm render
   ↓
6. Kiểm tra Kubernetes manifests
   ↓
7. Deploy Minikube
   ↓
8. Evidence → Decision


Runtime configuration flow

config/app/go-click-tracker.yaml
    ↓ configMap.data
Helm template: ConfigMap
    ↓ envFrom.configMapRef
StatefulSet Pod environment
    ↓
go-click-tracker process

`INSTANCE_ID` là ngoại lệ: nó lấy động từ `metadata.name` của Pod, không nằm
trong ConfigMap. Khi `configMap.data` thay đổi, checksum trong Pod template
thay đổi và StatefulSet được rollout để Pod nhận configuration mới.

Runtime configuration không chứa credential hoặc secret. Khi cần credential,
dùng `envFrom` với `secretRef` và lưu Secret qua cơ chế quản lý secret của
môi trường triển khai.




Nhiều input cùng đi vào Helm:

                   ┌─ Chart defaults
                    │  charts/statefulset/values.yaml
                    │
Application values ─┼─ config/app/go-click-tracker.yaml
                    │
Environment values ─┼─ config/environments/local/values.yaml
                    │
Release version ────┴─ config/environments/local/releases.yaml
                              ↓
                         Helm merge
                              ↓
             ConfigMap + Service + StatefulSet
                               ↓
                    Pod envFrom / runtime



Thứ tự ưu tiên:

Chart defaults
    < Application baseline
    < Environment override
    < Release-specific override


Với local go-click-tracker, lệnh render là:

    helm template go-click-tracker charts/statefulset \
      -f config/app/go-click-tracker.yaml \
      -f config/environments/local/values.yaml \
      -f config/environments/local/releases.yaml \
      --namespace local \
      > evidence/week-1/rendered-go-click-tracker.yaml

Khi cần đổi runtime configuration, sửa `configMap.data` trong file application
hoặc override theo environment, sau đó render/verify lại manifest. Không cần
build lại image chỉ vì đổi các giá trị configuration này.
