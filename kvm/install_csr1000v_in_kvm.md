# Installing CSR1000v in KVM using virt-install

jinja2 template っぽいのは特に意味なし

```yaml
qcow_iamge_path: /var/lib/libvirt/images/csr1000v.qcow2
csr1000v_iso_image_path: /home/myAccount/iso/csr1000v-targetVersion.iso
gi1_attached_bridge: mgmt-br
gi2_attached_bridge: right-br
gi3 attached_bridge: left-br
```

```
qemu-img create -f qcow2 "{{ qcow-image-path }}" 8G
virt-install \
    --connect=qemu:///system  \
    --name="{{ instance_name }}"  \
    --description "CSR1000v 3.14.02S"   \
    --os-type=linux  \
    --os-variant=rhel4  \
    --arch=x86_64 \
    --cpu host  \
    --vcpus=1,sockets=2,cores=1,threads=1   \
    --hvm \
    --ram=4096 \
    --cdrom="{{ csr1000v_iso_image_path}}" \
    --disk path=/var/lib/libvirt/images/csr1000v_GW.qcow2,bus=virtio,size=8,sparse=false,cache=none,format=qcow2   \
    --console pty,target_type=virtio  \
    --accelerate \
    --noautoconsole  \
    --video=vga  \
    --graphics vnc,listen=0.0.0.0 \
    --serial tcp,host=0.0.0.0:4576,mode=bind,protocol=telnet   \
    --network bridge="{{ gi1_attached_bridge }}",model=virtio  \
    --network bridge="{{ gi2_attached_brige }}",model=virtio  \
    --network bridge="{{ gi3_attached_brige }}",model=virtio \
    --noreboot
```

