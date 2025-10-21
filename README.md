# OpenShift GPU Time-Slicing Configuration

[English](#english) | [Español](#español)

---

## English

### 📋 Overview

This repository contains configuration files and documentation to enable GPU time-slicing on OpenShift with the NVIDIA GPU Operator. Time-slicing allows multiple pods to share a single physical GPU by time-multiplexing, maximizing GPU utilization for workloads that don't require exclusive GPU access.

### ✨ Features

- 🚀 Easy configuration with simple YAML manifests
- 📊 Multiply available GPU resources (e.g., 1 physical GPU → 2 logical GPUs)
- 🔧 Flexible replica configuration (2, 4, 8, or custom)
- 📚 Comprehensive documentation and troubleshooting guides
- 🧪 Ready-to-use test examples
- 🤖 OpenShift AI integration examples

### 🎯 Use Cases

**Ideal for:**
- Model inference workloads
- Development and testing environments
- Jupyter notebooks for data science
- CI/CD pipelines with GPU requirements
- Workloads with intermittent GPU usage

**Not recommended for:**
- Large model training requiring full GPU memory
- 24/7 intensive GPU workloads
- Applications requiring exclusive GPU access

### 📦 Requirements

- OpenShift 4.12+ (tested on 4.18)
- NVIDIA GPU Operator 24.9.0+ (tested on 24.9.2)
- At least one node with NVIDIA GPU
- cluster-admin permissions

### 🚀 Quick Start

#### 1. Clone the repository

```bash
git clone https://github.com/your-org/openshift-gpu-timeslicing.git
cd openshift-gpu-timeslicing
```

#### 2. Apply the ConfigMap

```bash
oc apply -f manifests/configmap/time-slicing-config.yaml
```

#### 3. Patch the ClusterPolicy

```bash
oc patch clusterpolicy gpu-cluster-policy -n nvidia-gpu-operator \
  --type=merge \
  --patch-file manifests/patches/clusterpolicy-patch.yaml
```

#### 4. Wait for device-plugin pods to restart

```bash
oc get pods -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset -w
```

Wait until all pods are in `Running` state.

#### 5. Verify GPU multiplication

```bash
# List GPU nodes
oc get nodes -l nvidia.com/gpu.present=true

# Check GPU resources on a node (should show replicas, e.g., 2 instead of 1)
oc describe node <gpu-node-name> | grep nvidia.com/gpu
```

### 🧪 Testing

Deploy test pods to verify time-slicing is working:

```bash
# Deploy two pods sharing the same GPU
oc apply -f manifests/examples/test-pod-multi.yaml

# Check both pods are running
oc get pods -l app=gpu-test-multi

# Verify GPU access from both pods
oc exec -it gpu-test-pod-1 -- nvidia-smi
oc exec -it gpu-test-pod-2 -- nvidia-smi
```

Both pods should show the same physical GPU.

### ⚙️ Configuration

#### Changing the number of replicas

Edit `manifests/configmap/time-slicing-config.yaml` and modify the `replicas` value:

```yaml
data:
  any: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 4  # Change this value (2, 4, 8, etc.)
```

Then reapply:

```bash
oc apply -f manifests/configmap/time-slicing-config.yaml
oc delete pods -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset
```

#### GPU-specific configurations

If you have different GPU types and want different configurations:

```yaml
data:
  a100: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 8
  t4: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 4
```

Then label your nodes:

```bash
oc label node <a100-node> nvidia.com/device-plugin.config=a100
oc label node <t4-node> nvidia.com/device-plugin.config=t4
```

### 🔍 Verification Commands

```bash
# Check ClusterPolicy status
oc get clusterpolicy gpu-cluster-policy -n nvidia-gpu-operator

# View device-plugin logs
oc logs -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset --tail=50

# Check GPU allocation on a node
oc describe node <gpu-node> | grep -A 10 "Allocated resources"

# View GPU capacity across all nodes
oc get nodes -l nvidia.com/gpu.present=true \
  -o custom-columns=NAME:.metadata.name,GPU:.status.capacity.nvidia\\.com/gpu
```

### 🧹 Cleanup

To remove time-slicing configuration:

```bash
# Remove the patch from ClusterPolicy
oc patch clusterpolicy gpu-cluster-policy -n nvidia-gpu-operator \
  --type=json \
  -p='[{"op": "remove", "path": "/spec/devicePlugin/config"}]'

# Delete the ConfigMap
oc delete configmap time-slicing-config -n nvidia-gpu-operator

# Restart device-plugin pods
oc delete pods -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset
```

### 📚 Documentation

- [Troubleshooting Guide](docs/troubleshooting.md) - Common issues and solutions
- [Architecture](docs/architecture.md) - How time-slicing works
- [Best Practices](docs/best-practices.md) - Recommendations and tips
- [FAQ](docs/faq.md) - Frequently asked questions

### ⚠️ Important Notes

- **GPU Memory**: Time-slicing does NOT divide GPU memory. If two pods try to use all available memory, OOM errors will occur.
- **Performance**: GPU processing time is shared. Heavy usage by one pod reduces available time for others.
- **MIG vs Time-Slicing**: For A100/H100 GPUs, consider MIG (Multi-Instance GPU) for hardware-level isolation instead of time-slicing.

### 🤝 Contributing

Contributions are welcome! Please open an issue or pull request.

### 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

### 🔗 Resources

- [NVIDIA GPU Operator Documentation](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/)
- [GPU Time-Slicing Guide](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html)
- [OpenShift Documentation](https://docs.openshift.com/)

---

## Español

### 📋 Descripción General

Este repositorio contiene archivos de configuración y documentación para habilitar time-slicing de GPUs en OpenShift con el NVIDIA GPU Operator. El time-slicing permite que múltiples pods compartan una sola GPU física mediante multiplexación temporal, maximizando la utilización de GPU para cargas de trabajo que no requieren acceso exclusivo.

### ✨ Características

- 🚀 Configuración fácil con manifiestos YAML simples
- 📊 Multiplica los recursos GPU disponibles (ej: 1 GPU física → 2 GPUs lógicas)
- 🔧 Configuración flexible de réplicas (2, 4, 8, o personalizado)
- 📚 Documentación completa y guías de solución de problemas
- 🧪 Ejemplos de prueba listos para usar
- 🤖 Ejemplos de integración con OpenShift AI

### 🎯 Casos de Uso

**Ideal para:**
- Cargas de trabajo de inferencia de modelos
- Entornos de desarrollo y pruebas
- Notebooks de Jupyter para ciencia de datos
- Pipelines de CI/CD con requisitos de GPU
- Cargas de trabajo con uso intermitente de GPU

**No recomendado para:**
- Entrenamiento de modelos grandes que requieren toda la memoria GPU
- Cargas de trabajo intensivas de GPU 24/7
- Aplicaciones que requieren acceso exclusivo a la GPU

### 📦 Requisitos

- OpenShift 4.12+ (probado en 4.18)
- NVIDIA GPU Operator 24.9.0+ (probado en 24.9.2)
- Al menos un nodo con GPU NVIDIA
- Permisos de cluster-admin

### 🚀 Inicio Rápido

#### 1. Clonar el repositorio

```bash
git clone https://github.com/your-org/openshift-gpu-timeslicing.git
cd openshift-gpu-timeslicing
```

#### 2. Aplicar el ConfigMap

```bash
oc apply -f manifests/configmap/time-slicing-config.yaml
```

#### 3. Parchear el ClusterPolicy

```bash
oc patch clusterpolicy gpu-cluster-policy -n nvidia-gpu-operator \
  --type=merge \
  --patch-file manifests/patches/clusterpolicy-patch.yaml
```

#### 4. Esperar a que los pods de device-plugin se reinicien

```bash
oc get pods -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset -w
```

Espera hasta que todos los pods estén en estado `Running`.

#### 5. Verificar la multiplicación de GPU

```bash
# Listar nodos con GPU
oc get nodes -l nvidia.com/gpu.present=true

# Verificar recursos GPU en un nodo (debería mostrar réplicas, ej: 2 en lugar de 1)
oc describe node <nombre-nodo-gpu> | grep nvidia.com/gpu
```

### 🧪 Pruebas

Despliega pods de prueba para verificar que time-slicing funciona:

```bash
# Desplegar dos pods compartiendo la misma GPU
oc apply -f manifests/examples/test-pod-multi.yaml

# Verificar que ambos pods están corriendo
oc get pods -l app=gpu-test-multi

# Verificar acceso a GPU desde ambos pods
oc exec -it gpu-test-pod-1 -- nvidia-smi
oc exec -it gpu-test-pod-2 -- nvidia-smi
```

Ambos pods deberían mostrar la misma GPU física.

### ⚙️ Configuración

#### Cambiar el número de réplicas

Edita `manifests/configmap/time-slicing-config.yaml` y modifica el valor de `replicas`:

```yaml
data:
  any: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 4  # Cambia este valor (2, 4, 8, etc.)
```

Luego vuelve a aplicar:

```bash
oc apply -f manifests/configmap/time-slicing-config.yaml
oc delete pods -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset
```

#### Configuraciones específicas por GPU

Si tienes diferentes tipos de GPU y quieres configuraciones diferentes:

```yaml
data:
  a100: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 8
  t4: |-
    version: v1
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 4
```

Luego etiqueta tus nodos:

```bash
oc label node <nodo-a100> nvidia.com/device-plugin.config=a100
oc label node <nodo-t4> nvidia.com/device-plugin.config=t4
```

### 🔍 Comandos de Verificación

```bash
# Verificar estado del ClusterPolicy
oc get clusterpolicy gpu-cluster-policy -n nvidia-gpu-operator

# Ver logs del device-plugin
oc logs -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset --tail=50

# Verificar asignación de GPU en un nodo
oc describe node <nodo-gpu> | grep -A 10 "Allocated resources"

# Ver capacidad de GPU en todos los nodos
oc get nodes -l nvidia.com/gpu.present=true \
  -o custom-columns=NAME:.metadata.name,GPU:.status.capacity.nvidia\\.com/gpu
```

### 🧹 Limpieza

Para eliminar la configuración de time-slicing:

```bash
# Eliminar el patch del ClusterPolicy
oc patch clusterpolicy gpu-cluster-policy -n nvidia-gpu-operator \
  --type=json \
  -p='[{"op": "remove", "path": "/spec/devicePlugin/config"}]'

# Eliminar el ConfigMap
oc delete configmap time-slicing-config -n nvidia-gpu-operator

# Reiniciar los pods de device-plugin
oc delete pods -n nvidia-gpu-operator -l app=nvidia-device-plugin-daemonset
```

### 📚 Documentación

- [Guía de Solución de Problemas](docs/troubleshooting.md) - Problemas comunes y soluciones
- [Arquitectura](docs/architecture.md) - Cómo funciona time-slicing
- [Mejores Prácticas](docs/best-practices.md) - Recomendaciones y consejos
- [Preguntas Frecuentes](docs/faq.md) - FAQ

### ⚠️ Notas Importantes

- **Memoria GPU**: Time-slicing NO divide la memoria de la GPU. Si dos pods intentan usar toda la memoria disponible, ocurrirán errores OOM.
- **Rendimiento**: El tiempo de procesamiento de la GPU se comparte. El uso intensivo por un pod reduce el tiempo disponible para otros.
- **MIG vs Time-Slicing**: Para GPUs A100/H100, considera MIG (Multi-Instance GPU) para aislamiento a nivel de hardware en lugar de time-slicing.

### 🤝 Contribuciones

¡Las contribuciones son bienvenidas! Por favor abre un issue o pull request.

### 📄 Licencia

Licencia MIT - ver archivo [LICENSE](LICENSE) para más detalles.

### 🔗 Recursos

- [Documentación de NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/)
- [Guía de GPU Time-Slicing](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html)
- [Documentación de OpenShift](https://docs.openshift.com/)