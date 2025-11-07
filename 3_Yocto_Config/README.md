# Cấu hình và build sản phẩm bằng Yocto
## [Yocto-BBB]  Lý thuyết, Tài liệu, và câu lệnh thường dùng trong Series Yocto
Khi thực hành theo các bài trong series, có những câu lệnh mình thường dùng, có 1 số lý thuyết có thể dễ quên, và 1 số tài liệu phụ. Ở bài này mình sẽ tổng hợp lại để cho các bạn dễ theo dõi, làm theo hơn .

### 1. Lý thuyết
```text
Yocto Project là gì ?
Poky là gì ?
Open Embedded là gì ?
Bitbake là gì ?
Recipes là gì ?
Layer là gì ?
File .bb, .bbappend, .bbclass 
```
### 2. Tài liệu

Reference Manual Beaglebone Black : https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf?ts=1757666524292&ref_url=https%253A%252F%252Fwww.google.com%252F

Yocto Project Reference Manual: https://docs.yoctoproject.org/ref-manual/index.html

[Xem tài liệu Yocto từ boolin](https://bootlin.com/doc/training/yocto/yocto-labs.pdf) (mình hay dùng từ trang chính thức của Bootlin)


### 3. Câu lệnh thường dùng
3.1 Yocto command
```bash
source poky/oe-init-build-end <build-dir>
#Khởi tạo môi trường build, add bitbake vào PATH  
```
```bash
bitbake <recipes>
# build <recipes>
```
```bash
bitbake -c <task> <recipes>
# build <recipes> từ đầu cho đến hết <task>
```
```bash
bitbake -e <recipes> | grep ^<Variable>
# lấy được các biến môi trường (env) của recipes
# grep dòng bắt đầu bởi <Variable>
```
```bash
bitbake-layers show-recipes
# Hiển thị địa chỉ toàn bộ recipes (file .bb) đang để tại meta-layer nào 
# Nếu chỉ đích danh <recipes> thì chỉ hiện meta-layer chứa nó
bitbake-layers show-recipes <recipes>
```
```bash
bitbake-layers show-appends <recipes>
# Tương tự với câu lệnh trên nhưng sẽ hiển thị địa chỉ file append (bbapend)
```
```bash
bitbake-layers show-layers 
# Hiển thị toàn bộ layers trong <build-dir>/conf/bbalyers.conf với Priority
```
```bash
bitbake-layers create-layer
# Tạo layer mới
```bash
```bash
bitbake-layers add-layer
# add layer tới bblayers.conf
```
```bash
oe-pkgdata-util lookup-recipe kernel
# Tim recipes kernel để các bạn có thể tạo file append
```
```bash
bitbake -c menuconfig virtual/kernel
# Bật menuconfig sửa kernel build config
```
### 3.2 Linux command
```bash
lsblk
# list block, dùng để check địa chỉ thẻ nhớ 
# ví dụ như /dev/sda, /dev/sdb, /dev/mmcblk0
```
```bash
bmaptool copy <yocto-build-image>.wic.xz <uSD disk>
# Flash thẻ nhớ với image, ví dụ như 
bmaptool copy core-image-minimal-beaglebone.wic.xz /dev/mmcblk0
```
```bash
find . -name <file-name>
# dùng để tìm kiếm đệ quy bắt đầu từ folder hiện tại tìm file-name
```
```bash
df -h 
# disk filesystem
# -h : Human readable, đổi sang hệ Gb, Mb, Kb cho ngừời dễ đọc 
```
```bash
free -h
# check dung lượng memory (RAM) 
```

---
---
## [Yocto-BBB] . Build Beaglebone Black Image Bằng Yocto Project
### Bắt đầu series về build bủng bằng Yocto thôi, bài đầu sẽ về build cho Beaglebone Black
![ảnh](https://embeddedlinux.blog/wp-content/uploads/2025/06/image-25.png?w=300)

1. Chuẩn bị bên máy host (Máy để build)
2. Clone source code
3. Setup và build
   - 3.1. Tạo môi trường build
    - 3.2 Thay đổi conf file
    -  3.3 Build thôi
4. Flash image SD card
     - 4.1 Check image
     -  4.2 Copy image sang sd card
### 1. Chuẩn bị bên máy host (Máy để build)
Ở đây máy mình cài Ubuntu 22.04.5 LTS (Jammy Jellyfish). Mình cài dual boot với windows để tối ưu core.

Dựa theo hướng dẫn bên Yocto Project Reference Manual , mục 1.3.2.1. Ta cài các package sau
```bash
sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
     build-essential chrpath socat cpio python3 python3-pip python3-pexpect \
     libsdl1.2-dev xterm make xsltproc docbook-utils fop dblatex xmlto
```
### 2. Clone source code
Chúng ta dùng bản poky là 1 repo mẫu từ bên Yocto Project (reference repo) với phiên bản Kirkstone 4.0 (Mình tính cài nhiều board với cùng folder Poky này nên cài bản này cho có tính tương thích cao hơn Scarthgap 5.0 mới ra)
```bash

git clone -b kirkstone git://git.yoctoproject.org/poky/
```
```bash
cd poky
 ```
 ```bash
git clone -b kirkstone git://git.openembedded.org/meta-openembedded
 ```
 ```bash
git clone -b kirkstone git://git.yoctoproject.org/meta-ti
 ```
 ```bash
git clone -b kirkstone https://git.yoctoproject.org/meta-arm
```
### 3. Setup và build
- 3.1. Tạo môi trường build
```bash
source oe-init-build-env build-bbb
```
Câu lệnh này sẽ
```text
Set $OEROOT : Xác định thư mục gốc của yocto
Set $PATH : Thêm bitbake vào path của session terminal, giống kiểu thêm vào biến môi trường ở Windows
Set $BUILDDIR : Chọn thư mục output ./build-bbb (default sẽ là build, nhưng nếu chạy cho nhiều board thì nên chia riêng)
Set $BBPATH : Để bitbake tìm thấy các layer
Tạo cấu trúc thư mục $BUILDDIR
```
![alt text](image.png)

- 3.2 Thay đổi conf file

   Thông thường ta sẽ thay đổi config file của môi trường build, bằng các layer, tuy nhiên ở đây để đơn giản mình sẽ thay đổi trong local.conf
```bash
nano ./build-bbb/conf/local.conf
```
Ta thay đổi MACHINE ?= “beaglebone”
```bash
nano ./build-bbb/conf/bblayers.conf
```
Ta add thêm các meta-openembedded, meta-arm, meta-ti, để có thêm các tính năng sẽ dùng sau
```bash
POKY_BBLAYERS_CONF_VERSION = "2"
 
BBPATH = "${TOPDIR}"
BBFILES ?= ""
 
BBLAYERS ?= " \
/home/zk47/Learning/poky/meta \
/home/zk47/Learning/poky/meta-poky \
/home/zk47/Learning/poky/meta-yocto-bsp \
/home/zk47/Learning/poky/meta-openembedded/meta-oe \
/home/zk47/Learning/poky/meta-arm/meta-arm-toolchain \
/home/zk47/Learning/poky/meta-arm/meta-arm \
/home/zk47/Learning/poky/meta-ti/meta-ti-bsp \
/home/zk47/Learning/poky/meta-ti/meta-ti-extras \
 
"
```
Lưu ý đường dẫn trong BBLAYERS là đường dẫn tuyệt đối tới các meta-layers, do đó sẽ tùy theo máy.

### 3.3 Build thôi
```bash
bitbake core-image-minimal
```
Ở đây mình chọn image này vì đúng như tên gọi, nó là cái tinh giản nhất, đúng chỉ là lên được OS, số lượng package rất it, nên build nhanh.

Tùy vào máy của các bạn, như máy mình build thời gian build rơi vào tầm hơn 1 tiếng.
![alt text](image-1.png)



### 4. Flash image SD card
- 4.1 Check image

Sau khi build xong nếu các bạn không biết output ở đâu có thể check bằng câu lệnh
```bash
bitbake -e core-image-minimal | grep ^WORKDIR=
```

- bitbake -e <recipe> : Liệt kê environment hay các biến được set trong 1 recipe
- grep ^WORKDIR= : tìm các dòng bắt đầu bằng “WORKDIR=” 

Output của câu lệnh sẽ dạng
```bash
~/Learning/poky/build-bbb$ bitbake -e core-image-minimal | grep ^WORKDIR=
WORKDIR="/home/zk47/Learning/poky/build-bbb/tmp/work/beaglebone-oe-linux-gnueabi/core-image-minimal/1.0-r0"
```
Các bạn vào được thư mục đó rồi mở tiếp folder deploy image sau cùng
```bash
cd /home/zk47/Learning/poky/build-bbb/tmp/work/beaglebone-oe-linux-gnueabi/core-image-minimal/1.0-r0
cd deploy-core-image-minimal-image-complete
```
4.2 Copy image sang sd card

Trong thư mục này sẽ có rất nhiều image
![alt text](image-2.png)


Nhưng để cho nhanh mình sẽ dùng image .wic.xz (tối ưu việc copy hơn do có bmap)

Các bạn cắm sd card vào, nếu đã có phân vùng được mount thì nhớ unmount trước, check bằng câu lệnh lsblk

![alt text](image-3.png)

Như ở đây thẻ nhớ mình đang nhận ở /dev/mmcblk0 và có 2 phân vùng đã được mount do đó ta cần umount nó trước
```bash
sudo umount /media/zk47/boot /media/bach/root
```
Sau đó ta copy sử dụng bmaptool
```bash
sudo bmaptool copy core-image-minimal-beaglebone.wic.xz /dev/mmcblk0
```
Và thế là xong, ta đã build thành công và flash vào thẻ nhớ. Giờ ta cắm thẻ nhớ vào BBB và test thôi.

Lưu ý giữ button S2 (boot button) để BBB nhận boot từ MMC1 là thẻ nhớ, nếu không giữ button S2 BBB sẽ boot từ MMC2 là eMMC.

![alt text](image-4.png)


–> Chúng ta sẽ đăng nhập bằng user root, không có mật khẩu

![alt text](image-5.png)

=> Quá trình tạo distro cho BBB bằng Yocto như vậy là done.

#  Hướng dẫn thêm, sửa, xóa Package trong Yocto Project

> Tài liệu hướng dẫn chi tiết cách tạo mới, chỉnh sửa và quản lý package (ứng dụng hoặc driver) trong hệ thống Yocto Project.  
> Đây là phần quan trọng giúp bạn **tùy biến Linux distro** theo nhu cầu thực tế.

---

##  Những thứ cần biết
1. [Giới thiệu tổng quan](#giới-thiệu-tổng-quan)
2. [Cấu trúc Layer trong Yocto](#cấu-trúc-layer-trong-yocto)
3. [Thêm mới Application Package](#thêm-mới-application-package)
4. [Thêm mới Kernel Driver](#thêm-mới-kernel-driver)
5. [Sửa và xóa package](#sửa-và-xóa-package)
6. [Tóm tắt quy trình](#tóm-tắt-quy-trình)
7. [Tài liệu tham khảo](#tài-liệu-tham-khảo)

---

## 1. Giới thiệu tổng quan

Trong **Yocto Project**, mỗi **package (gói)** được định nghĩa bởi **BitBake recipe** — các file `.bb` hoặc `.bbappend` nằm trong các **layer** (`meta-*`).

Các loại package thường gặp:
-  **Application (ứng dụng)** – ví dụ: `hello`, `nginx`
-  **Driver hoặc kernel module** – ví dụ: `my_usb_driver.ko`
-  **Library (thư viện)** – ví dụ: `libcurl`, `openssl`
-  **System component (thành phần hệ thống)** – ví dụ: `systemd`, `udev`

---

## 2. Cấu trúc Layer trong Yocto

Một layer có cấu trúc cơ bản như sau:
```text
meta-my-layer/
├── conf/
│ └── layer.conf
├── recipes-example/
│ └── hello/
│ ├── hello_1.0.bb
│ ├── files/
│ │ ├── hello.c
│ │ └── Makefile
└── README
```

> 💡 Mẹo: Tên layer nên bắt đầu bằng `meta-` để tuân thủ chuẩn của Yocto.

---

## 3. Thêm mới Application Package

###  Bước 1. Tạo layer mới (nếu chưa có)
```bash
bitbake-layers create-layer ../meta-my-layer
bitbake-layers add-layer ../meta-my-layer
```
 ### Bước 2. Tạo thư mục recipe
```bash
mkdir -p ../meta-my-layer/recipes-example/hello/files
cd ../meta-my-layer/recipes-example/hello/
```
### Bước 3. Viết mã nguồn ứng dụng
vd files/hello.c
```bash
#include <stdio.h>
int main() {
    printf("Hello Yocto!\n");
    return 0;
}
```
files/Makefile
```bash
all:
	$(CC) hello.c -o hello

install:
	mkdir -p $(DESTDIR)/usr/bin
	install -m 0755 hello $(DESTDIR)/usr/bin/
```
###  Bước 4. Tạo file recipe .bb
hello_1.0.bb
```bash
DESCRIPTION = "Simple Hello World Application"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COREBASE}/meta/files/common-licenses/MIT;md5=0835b3e998a47c4...your_md5_here..."

SRC_URI = "file://hello.c \
           file://Makefile"

S = "${WORKDIR}"

do_compile() {
    oe_runmake
}

do_install() {
    oe_runmake install DESTDIR=${D}
}

FILES:${PN} += "/usr/bin/hello"
```
### Bước 5. Build và kiểm tra
```bash
bitbake hello
```
Sau khi build thành công:

- File .ipk hoặc .rpm nằm trong:
```bash
tmp/deploy/ipk/<machine>/hello_1.0-r0.ipk
```
- Thêm package vào image:
```
IMAGE_INSTALL:append = " hello"
```
trong file local.conf hoặc core-image.bb.

4. Thêm mới Kernel Driver

-  Bước 1. Tạo thư mục recipe
```bash
mkdir -p ../meta-my-layer/recipes-kernel/mydriver/files
```
- Bước 2. Thêm mã nguồn driver
files/mydriver.c
```bash
#include <linux/module.h>
#include <linux/kernel.h>

static int __init mydriver_init(void) {
    pr_info("MyDriver: loaded!\n");
    return 0;
}

static void __exit mydriver_exit(void) {
    pr_info("MyDriver: unloaded!\n");
}

module_init(mydriver_init);
module_exit(mydriver_exit);
MODULE_LICENSE("GPL");
```

files/Makefile
```bash
obj-m += mydriver.o
```
-  Bước 3. Tạo recipe .bb
mydriver_1.0.bb
```bash
DESCRIPTION = "Custom kernel module example"
LICENSE = "GPLv2"
LIC_FILES_CHKSUM = "file://${COREBASE}/meta/files/common-licenses/GPL-2.0-only;md5=c72...md5"

inherit module

SRC_URI = "file://Makefile \
           file://mydriver.c"

S = "${WORKDIR}"

EXTRA_OEMAKE += "KERNELDIR=${STAGING_KERNEL_DIR}"

do_install() {
    install -d ${D}/lib/modules/${KERNEL_VERSION}/extra
    install -m 0644 mydriver.ko ${D}/lib/modules/${KERNEL_VERSION}/extra/
}

FILES:${PN} += "/lib/modules/${KERNEL_VERSION}/extra/mydriver.ko"
```
-  Bước 4. Build và thêm vào image
```bash
bitbake mydriver
```


Thêm driver vào image:
```bash
IMAGE_INSTALL:append = " mydriver"
```
5. Sửa và xóa package
 Sửa package
Nếu muốn mở rộng hoặc ghi đè recipe có sẵn → tạo file .bbappend tương ứng:
```bash
recipes-core/busybox/busybox_%.bbappend
```
Bên trong bạn có thể:

- Thêm patch

- Ghi đè SRC_URI

- Thay đổi do_install() hoặc do_compile()

 Xóa package
Gỡ khỏi IMAGE_INSTALL (nếu đã thêm)

Hoặc làm sạch build:
```bash
bitbake -c clean <package>
```
Hoặc comment trong local.conf

6. Tóm tắt quy trình

| Bước | Hành động                     | Lệnh chính                    |
| ---- | ----------------------------- | ----------------------------- |
| 1    | Tạo layer mới                 | `bitbake-layers create-layer` |
| 2    | Tạo recipe và file `.bb`      | `mkdir -p recipes-*`          |
| 3    | Viết code & Makefile          | —                             |
| 4    | Build thử package             | `bitbake <pkg>`               |
| 5    | Thêm vào image                | `IMAGE_INSTALL:append`        |
| 6    | Kiểm tra trên board hoặc QEMU | —                             |


7. Tài liệu tham khảo
> Yocto Project Development Manual

> Yocto Reference Manual

> BitBake User Manual

> OpenEmbedded Layer Index
