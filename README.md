# Configuration of the smart home controller Pi 4

This should probably be an Ansible config, or just done automagically by cloud-init

## After writing image

* Copy the system-boot directory content into the system-boot partition mount point

* Configure zswap by appending to /system-boot/cmdline.txt:

      zswap.enabled=1 zswap.compressor=lz4 zswap.max_pool_percent=20 zswap.zpool=z3fold

## After first boot

* SSH to the new instance

* Extract our files

      sudo tar -C writable | pv | zstd | ssh ubuntu@10.0.0.4 sudo tar -C / --numeric-owner --zstd -x

* Allocate swap file:

      sudo fallocate /swapfile --length 2G
      sudo chmod 0600 /swapfile
      sudo mkswap /swapfile
      echo '/swapfile  swap swap  defaults  0 2' | sudo tee -a /etc/fstab
      sudo swapon -a

* Enable our services:

      sudo systemctl enable --now dwd_10mins.timer
      sudo systemctl enable --now wg-quick@wg0.service
      sudo systemctl enable --now home-assistant.service
      sudo systemctl enable --now unifi-controller.service
      sudo systemctl enable --now iperf3-server@5201.service
