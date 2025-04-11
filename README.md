# terraform-pfsense-kvm-libvirt

Automatiza el despliegue de **pfSense** como máquina virtual en **KVM/QEMU** utilizando **Terraform**, **libvirt**, y una **imagen base preinstalada en formato QCOW2**.

Ideal para entornos de laboratorio, pruebas de infraestructura o firewalls virtualizados. Incluye integración con el proveedor oficial de Terraform para pfSense (`marshallford/pfsense`), VNC, y soporte para configuración de red personalizada.

---

## 📦 Tecnologías usadas

- Terraform `>= 1.11.3`
- Provider: [`libvirt`](https://github.com/dmacvicar/terraform-provider-libvirt)
- Provider: [`pfsense`](https://registry.terraform.io/providers/marshallford/pfsense/latest)
- KVM + QEMU + virsh
- Imagen base: `pfsense_base_optimized.qcow2` (no ISO)

---

## 🚀 Despliegue rápido

1. Crea una imagen base de pfSense siguiendo los pasos en [`doc/pfsense_base_qcow2.md`](doc/pfsense_base_qcow2.md)
2. Clona este repositorio
3. Ajusta `terraform.tfvars` con las rutas y configuración necesarias
4. Ejecuta:

```bash
terraform init
terraform apply
```

---

## 📂 Estructura del repositorio

```bash
terraform-pfsense-kvm-libvirt/
├── main.tf
├── outputs.tf
├── provider.tf
├── terraform.tfvars
├── variables.tf
├── config/
│   ├── network-config.yaml
│   └── pfsense-user-data.yaml
├── doc/
│   └── pfsense_base_qcow2.md
```

---

## 🔧 Configuración vía `terraform.tfvars`

```hcl
pfsense_image     = "/mnt/lv_data/organized_storage/images/pfsense_base_optimized.qcow2"
pfsense_pool_path = "/mnt/lv_data/organized_storage/volumes/pfsense"

pfsense_vm_config = {
  cpus         = 2
  memory       = 2048
  disk_size_gb = 40
  wan_ip       = "192.168.0.200"
  lan_ip       = "192.168.1.1"
}

pfsense_vm_name = "pfsense_net"
```

---

## 🌐 Configuración de red

- **Bridge WAN:** `br0` → IP pública o red local (`192.168.0.200`)
- **Bridge LAN:** `br1` → Red interna o segmentada (`192.168.1.1`)
- Rutas manuales en Windows (si usas la LAN de pfSense):

```bash
route add 192.168.1.0 mask 255.255.255.0 192.168.0.200
```

---

## 📁 Configuración cloud-init *(opcional)*

Puedes añadir claves SSH y red DHCP por `cloud-init` usando:

- `config/network-config.yaml`
- `config/pfsense-user-data.yaml`

---

## 🛠 Archivos útiles

- [`doc/pfsense_base_qcow2.md`](doc/pfsense_base_qcow2.md): Cómo optimizar, copiar y usar tu imagen base
- Configuración DNS en `resolv.conf` y ajustes de paquetes en pfSense

---

## 🧪 Extras

- Acceso VNC automático: se habilita para acceder a la VM gráficamente
- La imagen base debe tener ya configurado:
  - Acceso SSH si se desea
  - Interfaces `vtnet0` y `vtnet1` para WAN/LAN
  - IPs estáticas opcionales

---

## 🔒 Recomendaciones

- Usa esta solución solo en entornos cerrados o de pruebas.
- pfSense está diseñado para uso con políticas de seguridad estrictas.
- Protege tu VNC y red de acceso si expones la VM.

---

## 📜 Licencia

MIT © 2025 – Basado en tecnologías Open Source.

---
