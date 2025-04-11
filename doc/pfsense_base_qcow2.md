# Guía para optimizar y preparar la imagen base QCOW2 de pfSense

Este documento describe cómo descargar, instalar, optimizar y preparar una imagen QCOW2 de pfSense para su uso en despliegues automatizados con Terraform y libvirt.

---

## 🌐 0. Descargar la ISO de pfSense

1. Ve al sitio oficial de descargas de Netgate:  
   👉 https://www.pfsense.org/download/

2. Selecciona:
   - **Architecture**: AMD64 (64-bit)
   - **Installer**: DVD Image (ISO)
   - **Console**: VGA
   - Descarga la ISO y colócala en una ruta como `/var/lib/libvirt/boot/`.

```bash
mkdir -p /var/lib/libvirt/boot/
mv ~/Downloads/pfSense-CE-*.iso /var/lib/libvirt/boot/pfsense.iso
```

---

## 🖥️ 1. Crear una máquina virtual para instalar pfSense

Crea una VM con `virt-manager`, `virt-install` o Terraform para instalar pfSense de forma manual:

```bash
virt-install \
  --name pfsense-install \
  --memory 2048 \
  --vcpus 2 \
  --disk size=10,format=qcow2,path=/var/lib/libvirt/images/pfsense_base.qcow2 \
  --cdrom /var/lib/libvirt/boot/pfsense.iso \
  --os-type=generic \
  --network bridge=br0 \
  --network bridge=br1 \
  --graphics vnc,listen=0.0.0.0
```

Sigue los pasos del instalador de pfSense y completa la instalación con IPs estáticas si lo deseas. Luego apaga la VM.

---

## 📌 2. Optimizar la imagen original

Usa `qemu-img` para crear una versión optimizada de la imagen base. Esto reduce espacio sin modificar la original:

```bash
qemu-img convert -O qcow2 /var/lib/libvirt/images/pfsense_base.qcow2 /var/lib/libvirt/images/pfsense_base_optimized.qcow2
```

---

## 📏 3. Verificar tamaños

Compara los tamaños de ambas imágenes:

```bash
ls -lh /var/lib/libvirt/images/pfsense_base.qcow2
ls -lh /var/lib/libvirt/images/pfsense_base_optimized.qcow2
```

---

## 📂 4. Mover imágenes a ubicación centralizada

```bash
# Imagen original
sudo cp /var/lib/libvirt/images/pfsense_base.qcow2 /mnt/lv_data/organized_storage/images/pfsense_base.qcow2

# Imagen optimizada
sudo cp /var/lib/libvirt/images/pfsense_base_optimized.qcow2 /mnt/lv_data/organized_storage/images/pfsense_base_optimized.qcow2
```

---

## 🔐 5. Asignar permisos adecuados

```bash
sudo chown qemu:qemu /mnt/lv_data/organized_storage/images/pfsense_base*.qcow2
sudo chmod 775 /mnt/lv_data/organized_storage/images/pfsense_base*.qcow2
```

---

## ✅ 6. Verificación final

```bash
ls -lh /mnt/lv_data/organized_storage/images/pfsense_base*.qcow2
```

---

## 🔄 7. Reiniciar el servicio SSH

```bash
service sshd restart
```

---

## 📦 8. Configurar repositorios en pfSense

```bash
vi /usr/local/etc/pkg/repos/pfSense.conf
```

Contenido:

```text
FreeBSD: { enabled: no }

pfSense-core: {
  url: "https://pkg.pfsense.org/pfSense_v2_7_2_amd64-core",
  mirror_type: "none",
  signature_type: "fingerprints",
  fingerprints: "/usr/local/share/pfSense/keys/pkg",
  enabled: yes
}

pfSense: {
  url: "https://pkg.pfsense.org/pfSense_v2_7_2_amd64-pfSense_v2_7_2",
  mirror_type: "none",
  signature_type: "fingerprints",
  fingerprints: "/usr/local/share/pfSense/keys/pkg",
  enabled: yes
}
```

```bash
certctl rehash
pkg-static clean -ay
pkg-static install -fy pkg pfSense-repo pfSense-upgrade
```

---

## 🌐 9. Configurar DNS (opcional)

```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

O múltiples entradas:

```text
nameserver 127.0.0.1
nameserver 8.8.8.8
nameserver 8.8.4.4
```

---

## 🌉 10. Asignar IP LAN manualmente

```bash
ifconfig vtnet1 inet 192.168.1.1 netmask 255.255.255.0 up
```

---

## 🪟 11. Ruta estática en Windows (opcional)

```cmd
route add 192.168.1.0 mask 255.255.255.0 192.168.0.200
```

---

## 🏁 Conclusión

Tu imagen base optimizada de pfSense está lista para ser usada en despliegues con Terraform + libvirt.

---
