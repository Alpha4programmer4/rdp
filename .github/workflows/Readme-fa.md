
## سرورهای عمومی رایگان

شما مي‌توانید از سرورهای زیر به رایگان استفاده کنید. این لیست ممکن است به مرور زمان تغییر می‌کند. اگر به این سرورها نزدیک نیستید، ممکن است اتصال شما کند باشد.
| موقعیت | سرویس دهنده | مشخصات |
| --------- | ------------- | ------------------ |
| آلمان | Hetzner | 2 vCPU / 4GB RAM |
| آلمان | Codext | 4 vCPU / 8GB RAM |

## وابستگی ها

نسخه‌های رومیزی از [sciter](https://sciter.com/) برای رابط کاربری گرافیکی استفاده می‌کنند. خواهشمندیم کتابخانه‌ی پویای sciter را خودتان دانلود کنید از این منابع دریافت کنید.

- [ویندوز](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.win/x64/sciter.dll)
- [لینوکس](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.lnx/x64/libsciter-gtk.so)
- [مک](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.osx/libsciter.dylib)

نسخه های همراه از Flutter استفاده می کنند. نسخه‌ی رومیزی را هم از Sciter به Flutter منتقل خواهیم کرد.

## نیازمندی‌های ساخت

- محیط توسعه نرم افزار Rust و محیط ساخت ++C خود را آماده کنید

- نرم افزار [vcpkg](https://github.com/microsoft/vcpkg) را نصب کنید و متغیر `VCPKG_ROOT` را به درستی تنظیم کنید.
- بسته‌های vcpkg مورد نیاز را نصب کنید:
  - ویندوز: `vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static`
  - مک و لینوکس: `vcpkg install libvpx libyuv opus aom`
- این دستور را اجرا کنید: `cargo run`

## [ساخت](https://rustdesk.com/docs/en/dev/build/)

## نحوه ساخت بر روی لینوکس

### ساخت بر روی (Ubuntu 18 (Debian 10

```sh
sudo apt install -y g++ gcc git curl wget nasm yasm libgtk-3-dev clang libxcb-randr0-dev libxdo-dev libxfixes-dev libxcb-shape0-dev libxcb-xfixes0-dev libasound2-dev libpulse-dev cmake
```

### ساخت بر روی (Fedora 28 (CentOS 8

```sh
sudo yum -y install gcc-c++ git curl wget nasm yasm gcc gtk3-devel clang libxcb-devel libxdo-devel libXfixes-devel pulseaudio-libs-devel cmake alsa-lib-devel
```

### ساخت بر روی (Arch (Manjaro

```sh
sudo pacman -Syu --needed unzip git cmake gcc curl wget yasm nasm zip make pkg-config clang gtk3 xdotool libxcb libxfixes alsa-lib pipewire
```

### نرم افزار vcpkg را نصب کنید

```sh
git clone https://github.com/microsoft/vcpkg
cd vcpkg
git checkout 2023.04.15
cd ..
vcpkg/bootstrap-vcpkg.sh
export VCPKG_ROOT=$HOME/vcpkg
vcpkg/vcpkg install libvpx libyuv opus aom
```

### رفع ایراد libvpx (برای فدورا)

```sh
cd vcpkg/buildtrees/libvpx/src
cd *
./configure
sed -i 's/CFLAGS+=-I/CFLAGS+=-fPIC -I/g' Makefile
sed -i 's/CXXFLAGS+=-I/CXXFLAGS+=-fPIC -I/g' Makefile
make
cp libvpx.a $HOME/vcpkg/installed/x64-linux/lib/
cd
```

### ساخت

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
git clone https://github.com/rustdesk/rustdesk
cd rustdesk
mkdir -p target/debug
wget https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.lnx/x64/libsciter-gtk.so
mv libsciter-gtk.so target/debug
VCPKG_ROOT=$HOME/vcpkg cargo run
```

### تغییر Wayland به (X11 (Xorg

راست‌دسک از Wayland پشتیبانی نمی کند. برای جایگزنی Xorg به عنوان پیش‌فرض GNOM، [اینجا](https://docs.fedoraproject.org/en-US/quick-docs/configuring-xorg-as-default-gnome-session/) را کلیک کنید.

## نحوه ساخت با داکر

این مخزن Git را دریافت کنید و کانتینر را به روش زیر بسازید

```sh
git clone https://github.com/rustdesk/rustdesk
cd rustdesk
docker build -t "rustdesk-builder" .
```

سپس، هر بار که نیاز به ساخت نرم‌افزار داشتید، دستور زیر را اجرا کنید:

```sh
docker run --rm -it -v $PWD:/home/user/rustdesk -v rustdesk-git-cache:/home/user/.cargo/git -v rustdesk-registry-cache:/home/user/.cargo/registry -e PUID="$(id -u)" -e PGID="$(id -g)" rustdesk-builder
```

توجه داشته باشید که نخستین ساخت ممکن است به دلیل محلی نبودن وابستگی‌ها بیشتر طول بکشد. اما دفعات بعدی سریعتر خواهند بود. علاوه بر این، اگر نیاز به تعیین آرگومان های مختلف برای دستور ساخت دارید، می توانید این کار را در انتهای دستور ساخت و از طریق `<OPTIONAL-ARGS>` انجام دهید. به عنوان مثال، اگر می خواهید یک نسخه نهایی بهینه سازی شده ایجاد کنید، دستور بالا را تایپ کنید و در انتها  `release--` را اضافه کنید. فایل اجرایی به دست آمده در پوشه مقصد در سیستم شما در دسترس خواهد بود و می تواند با دستور:

```sh
target/debug/rustdesk
```

یا برای نسخه بهینه سازی شده دستور زیر را اجرا کنید:

```sh
target/release/rustdesk
```

لطفاً اطمینان حاصل کنید که این دستورات را از پوشه مخزن RustDesk اجرا می کنید، در غیر این صورت ممکن است برنامه نتواند منابع مورد نیاز را پیدا کند. همچنین توجه داشته باشید که سایر دستورات فرعی Cargo مانند `install` یا `run` در حال حاضر از طریق این روش پشتیبانی نمی شوند زیرا برنامه به جای سیستم عامل میزبان, در داخل کانتینر نصب و اجرا میشود.

## ساختار پوشه ها 

- **[libs/hbb_common](https://github.com/rustdesk/rustdesk/tree/master/libs/hbb_common)**: video codec, config, tcp/udp wrapper, protobuf, fs functions for file transfer, and some other utility functions
- **[libs/scrap](https://github.com/rustdesk/rustdesk/tree/master/libs/scrap)**: screen capture
- **[libs/enigo](https://github.com/rustdesk/rustdesk/tree/master/libs/enigo)**: platform specific keyboard/mouse control
- **[src/ui](https://github.com/rustdesk/rustdesk/tree/master/src/ui)**: GUI
- **[src/server](https://github.com/rustdesk/rustdesk/tree/master/src/server)**: audio/clipboard/input/video services, and network connections
- **[src/client.rs](https://github.com/rustdesk/rustdesk/tree/master/src/client.rs)**: start a peer connection
- **[src/rendezvous_mediator.rs](https://github.com/rustdesk/rustdesk/tree/master/src/rendezvous_mediator.rs)**: Communicate with [rustdesk-server](https://github.com/rustdesk/rustdesk-server), wait for remote direct (TCP hole punching) or relayed connection
- **[src/platform](https://github.com/rustdesk/rustdesk/tree/master/src/platform)**: platform specific code
- **[flutter](https://github.com/rustdesk/rustdesk/tree/master/flutter)**: Flutter code for mobile
- **[flutter/web/js](https://github.com/rustdesk/rustdesk/tree/master/flutter/web/js)**: Javascript for Flutter web client

## تصاویر محیط نرم‌افزار

![image](https://user-images.githubusercontent.com/71636191/113112362-ae4deb80-923b-11eb-957d-ff88daad4f06.png)

![image](https://user-images.githubusercontent.com/71636191/113112619-f705a480-923b-11eb-911d-97e984ef52b6.png)

![image](https://user-images.githubusercontent.com/71636191/113112857-3fbd5d80-923c-11eb-9836-768325faf906.png)

![image](https://user-images.githubusercontent.com/71636191/135385039-38fdbd72-379a-422d-b97f-33df71fb1cec.png)
