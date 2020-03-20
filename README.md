# Kubernetes Klaster Qurulum (Ansible vasitəsilə)
Bu repozitory, kubetnetes klaster qurulumunu avtomatlaşdırmaq üçün ansible playbook-ları özündə birləşdirir. Repozitori virtual, fiziki və ya bulud texnologiyalarında yerləşən serverlər üzərinə icra edilə bilər. Qurulum, Centos8 distributivi üzərində testləşdirilib. Hər 3 serverin global şəbəkəyə (internet) çıxışı təmin edilməlidir. Qurulum hər 3 serverdə "root" istifadəçisi vasitəsilə aparılacaq (əlavə sazlamalar aparmaqla adi istifadəçi vasitəsilə də reallaşdırmaq olar). Qurulum üçün 3 ədəd Linux distributivi (Centos8) quraşdırılmış serverə ehtiyac var.  Klaster, 1 ədəd master və 2   ədəd worker tipli serverdən ibarət olacaq. Avtomatlaşdırma üçün ansible proqram təminatı master node üzərində quraşdırılacaq və özü də daxil olmaqla digər nodeları idarə edəcək(Klasterə sonradan worker tipli node əlavə edilməsi üçün ayrıca playbook repozitoridə yerləşdirilmişdir). Host adları və ip ünvanları xüsusiləşdirilə bilər:
   master
   worker1
   worker2

İnstruksiya:

1. Master node üzərində "git" proqram təminatı vasitəsilə qurulum üçün istifadə ediləcək repozitory yüklənir (əgər master node-da "git" yoxdursa, əvvəlcə onu quraşdırmaq gərəkdir):
   [root@master ~]# 

2. Serverlərin bir-birini görə bilməsi üçün host fayllarda (və ya DNS) uyğun sazlamaların aparılması və ip adreslərin host adları ilə adlandırılır:
   [root@master ~]# cat << /etc/hosts >> EOF
    > 10.1.31.13		master
    > 10.1.31.14		worker1
    > 10.1.31.15		worker2
  [root@worker1 ~]# cat << /etc/hosts >> EOF
    > 10.1.31.13		master
    > 10.1.31.14		worker1
    > 10.1.31.15		worker2
  [root@worker2 ~]# cat << /etc/hosts >> EOF
    > 10.1.31.13		master
    > 10.1.31.14		worker1
    > 10.1.31.15		worker2

3. Master node üzərində ssh-key yaradılaraq bütün nodelar ilə ssh vasitəsilə şifrəsiz qoşulma imkanı yaradılmalıdır:
   [root@master ~]# ssh-keygen
   [root@master ~]# ssh-copy-id master
   [root@master ~]# ssh-copy-id worker1
   [root@master ~]# ssh-copy-id worker2

4. Master node üzərində ansible proqram təminatı quraşdırılmalı
   [root@master ~]# dnf install -y epel-release
   [root@master ~]# dnf install -y ansible
   

Bundan sonra master nodedan ansible vasitəsilə hər 3 nodeun idarəedilmə imkanının olması yoxlanılmalıdır (# ansible all -m ping).
Repozitori master node vasitəsilə yükləndikdən sonra, ilk öncə hosts faylında uyğun sazlamalar aparılaraq master və worker nodeların adlarını xüsusiləşdirmək gərəkdir. Bundan sonra "vars_file" faylında sazlama aparılaraq klasterdə yerləşdiriləcək pod-lar üçün şəbəkə təyinatı pods_network parametrinə qiymət təyin etməklə aparmaq olar. Sazlamalar bitdikdən sonra "ansible-playbook k8s-install.yml" komandasını icra etməklə klasterin qurulumunu icra etmək olar. Qurulum uğurla bitdikdən sonra master node üzərində "kubectl get nodes" komandası vasitəsilə klasterin cari vəziyyətini yoxlamaq mümkündür.