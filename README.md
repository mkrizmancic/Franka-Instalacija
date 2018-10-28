# Franka Panda - upute za postavljanje računala

## Uvod
Franka Panda robotska ruka komunicira s upravljačkim računalom na frekvenciji od 1 kHz. Kako bi se održala ta frekvencija, na računalu mora biti postavljen *real-time kernel* te instalirani potrebni paketi. Ove upute sadrže sve korake potrebne za postavljanje računala koje se može koristiti za upravljanje Franka Panda rukom.

Franka Emika je izdala i svoje službene [upute za postavljanje](https://frankaemika.github.io/docs/overview.html). Te upute su vrlo kvalitetne i detaljne pa će i ovdje često biti referencirane. Preporuka je pratiti ih paralelno uz ove upute jer će ovdje biti navedene samo razlike koje su uočene tijekom instalacije te poneki savjet.

## 1. Instalacija operacijskog sustava
Operacijski sustav koji ćemo koristiti je **Linux Ubuntu 16.04. LTS**.<br>
Najnovija verzija je `16.04.5` koja dolazi s kernelom `v4.15`. Nju je moguće preuzeti [ovdje](http://releases.ubuntu.com/xenial/) (pri dnu stranice).<br>
Također je moguće koristiti stariju verziju: Ubuntu `16.04.4` s kernelom `v4.13`. Ovu verziju je moguće preuzeti [ovdje](http://old-releases.ubuntu.com/releases/16.04.4/).<br>
Operacijski sustav instalirati standardnim postupkom.<br>
**Prilikom instalacije kreirati i u nastavku koristiti *admin* račun.**

**Zašto koristiti stariju verziju?**<br>
*Real-time* kernel dolazi u obliku patcha za osnovnu verziju kernela. Patchevi postoje samo za neke verzije i poželjno je da se one podudaraju. Ako ne postoji patch za određenu verziju, treba koristiti onaj koji je najbliži verziji osnovnog kernela. ([izvor](https://frankaemika.github.io/docs/installation.html#setting-up-the-real-time-kernel)) Najnovija verzija patcha za kernel `v4.13` je `4.13.13-rt5`. Patch za kernel `v4.15` ne postoji, a najbliža verzija je `4.16.7-rt1`. Ovdje je korištena druga opcija i nisu uočeni nikakvi "system-breaking" problemi.<br>
Sve dostupne verzije *real-time* patcheva mogu se pronaći [ovdje](https://www.kernel.org/pub/linux/kernel/projects/rt/).

## 2. Postavljanje real-time kernela
Nakon što smo instalirali linux, treba dodati *real-time* patch.

Prvo treba instalirati sve preduvjete:<br>
(dodani su neki paketi u odnosu na službene upute)

    sudo apt-get install build-essential bc curl ca-certificates fakeroot gnupg2 libssl-dev lsb-release libelf-dev libssl1.1 bison flex

Sljedeće što treba napraviti je preuzeti *real-time patcheve* koji će odgovarati instaliranoj verziji kernela kao što je prije objašnjeno. Za ovaj korak prvo pročitati [službenu dokumentaciju](https://frankaemika.github.io/docs/installation.html#setting-up-the-real-time-kernel).<br>
Stvorimo novi radni folder:
    
    cd ~
    mkdir rt_kernel
    cd rt_kernel

Pretpostavka je da je verzija osnovnog kernela `4.15.0`, a patch `4.16.7-rt1`.<br>
Koristimo `curl` za preuzimanje potrebnih datoteka:

    curl -SLO https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.15.tar.xz
    curl -SLO https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.15.tar.sign
    curl -SLO https://www.kernel.org/pub/linux/kernel/projects/rt/4.14/older/patch-4.16.7-rt1.patch.xz
    curl -SLO https://www.kernel.org/pub/linux/kernel/projects/rt/4.14/older/patch-4.16.7-rt1.patch.sign
    
Zatim ih raspakiramo:

    xz -d linux-4.15.tar.xz
    xz -d patch-4.16.7-rt1.patch.xz
    
### Provjera preuzetih datoteka
Prije početka instalacije, dobro je provjeriti jesu li datoteke preuzete ispravno. Za ovaj korak se možemo poslužiti [službenom dokumentacijom](https://frankaemika.github.io/docs/installation.html#verifying-file-integrity).

### Kompajliranje kernela
Ako su sve preuzete datoteke u redu, možemo nastaviti. I ovaj dio prati [službenu dokumentaciju](https://frankaemika.github.io/docs/installation.html#compiling-the-kernel).

Raspakiramo *source code* i primijenimo patch (**pripaziti na ispravnu verziju!**):

    tar xf linux-4.15.tar
    cd linux-4.15
    patch -p1 < ../patch-4.16.7-rt1.patch
    
Zatim treba postaviti opcije za kernel:

    make oldconfig
    
Ovdje će biti jako puno opcija koje je preporučljivo ostaviti na default vrijednosti. Default vrijednost se prihvaća kada je pritisnut **enter** bez prethodnog unosa. Jedina opcija koju moramo promijeniti je sljedeća:

    Preemption Model
        1. No Forced Preemption (Server) (PREEMPT_NONE)
        2. Voluntary Kernel Preemption (Desktop) (PREEMPT_VOLUNTARY)
        3. Preemptible Kernel (Low-Latency Desktop) (PREEMPT__LL) (NEW)
        4. Preemptible Kernel (Basic RT) (PREEMPT_RTB) (NEW)
        > 5. Fully Preemptible Kernel (RT) (PREEMPT_RT_FULL) (NEW)

Zatim pokrećemo kompajlaciju kernela. Službene upute koriste `fakeroot` naredbu. To je stvaralo probleme prilikom instalacije jer s admin računom već imamo `root` pristup. Preporuka da se ona izostavi, a proces pokrene s:

    make -j4 deb-pkg
    
Nakon što proces kompajlacije završi, treba instalirati upravo stvorene pakete:

    sudo dpkg -i ../linux-headers-4.16.7-rt1_*.deb ../linux-image-4.16.7-rt1_*.deb
    
### Provjera kernela
Možemo provjeriti je li instalacija *real-time* kernela uspješno izvršena prema uputama u [službenoj dokumentaciji](https://frankaemika.github.io/docs/installation.html#verifying-the-new-kernel).

### Dodavanje korisničkih grupa
Posljednji korak je omogućiti korisniku postavljanje *real-time* dopuštenja za procese. Ponovno, koraci su navedeni u [službenoj dokumentaciji](https://frankaemika.github.io/docs/installation.html#allow-a-user-to-set-real-time-permissions-for-its-processes).

### Preventivne mjere za bolje performanse
Isključiti skaliranje CPU frekvencije prema [uputama](https://frankaemika.github.io/docs/troubleshooting.html#disabling-cpu-frequency-scaling).

## 3. Instalacija ROS-a
Franka koristi ROS Kinetic. Instalirati prema [uputama](http://wiki.ros.org/kinetic/Installation/Ubuntu).

## 4. Instalacija potrebnih paketa
Na kraju, potrebno je instalirati `libfranka` C++ api i `franka_ros` paket.<br>
Oboje je dostupno u ROS repozitorijima:

    sudo apt install ros-kinetic-libfranka ros-kinetic-franka-ros
    
## 5. Kontakt
Prva instalacija je provedena na Lenovo ThinkPad T520 laptopu (INV_BR: 40, FER: 126883, bivši Hausov laptop).

Instalacija i upute by Marko Križmančić (marko.krizmancic@fer.hr, github: mkrizmancic)
