# Kubernetes Klaster Qurulum (Ansible vasitəsilə)
Bu repozitori, kubetnetes klaster qurulumunu avtomatlaşdırmaq üçün ansible playbook-ları özündə birləşdirir. Repozitori vasitəsilə k8s qurulumu virtual, fiziki və ya bulud texnologiyalarında yerləşən serverlər üzərinə icra edilə bilər. Qurulum üçün 3 ədəd Linux distributivi (Centos8) quraşdırılmış serverə ehtiyac var. Klaster, 1 ədəd master və 2 ədəd worker tipli serverdən ibarət olacaq. Hər 3 serverin global şəbəkəyə (internet) çıxışı təmin edilməlidir. Qurulum hər 3 serverdə "root" istifadəçisi vasitəsilə aparılacaq (əlavə sazlamalar aparmaqla adi istifadəçi vasitəsilə də reallaşdırmaq olar). Avtomatlaşdırma üçün ansible proqram təminatı master node üzərində quraşdırılacaq və özü də daxil olmaqla digər nodeları idarə edəcək (Klasterə sonradan worker tipli node əlavə edilməsi üçün ayrıca playbook repozitoridə yerləşdirilmişdir). Qurulum, Centos8 distributivi üzərində testləşdirilib. Host adları və ip ünvanları xüsusiləşdirilə bilər:

   master
   
   worker1
   
   worker2

İnstruksiya:

1. Master node üzərində "git" proqram təminatı vasitəsilə qurulum üçün istifadə ediləcək repozitory yüklənir (əgər master node-da "git" yoxdursa, əvvəlcə onu quraşdırmaq gərəkdir):

   [root@master ~]# dnf -install -y git
   
   [root@master ~]# git clone https://github.com/anargurbanli/k8s-ansible

2. Serverlərin bir-birini görə bilməsi üçün host fayllarda (və ya DNS serverdə) uyğun sazlamaların aparılır və ip adreslər host adları ilə adlandırılır:
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

4. Master node üzərində ansible proqram təminatı quraşdırılmalıdır:

   [root@master ~]# dnf install -y epel-release
   
   [root@master ~]# dnf install -y ansible
   
5. Node-ların idarəolunmasını yoxlamaq üçün, master node-da yüklənmiş repozitoriyə daxil olmaq və "hosts" faylında sazlamalar apararaq master və worker node-ların adlarını düzgün qeyd etmək gərəkdir:

[root@master ~]# cd ~/k8s-ansible

[root@master k8s-ansible]# vi hosts

6. Master node-un ansible vasitəsilə bütün nodeları idarəetmə imkanı yoxlanılır:

   [root@master k8s-ansible]# ansible all -m ping

7. Master node-da cari qovluqda yerləşən "vars_file" faylında pods_network parametrinə qiymət təyin etməklə, klasterdə yaradılacaq pod-lar üçün şəbəkə təyin edilməsini xüsusiləşdirmək mümkündür.

8. Master node-da cari qovluqda "k8s-install.yml" playbook-u işə salınır və qurulum icra olunur:

  [root@master k8s-ansible]# ansible-playbook k8s-install.yml

9. Qurulum bitdikdən sonra master node üzərində "kubectl" komandası vasitəsilə klasterin cari vəziyyəti yoxlanılır:

  [root@master k8s-ansible]# kubectl get nodes

10. Klasterə node əlavə edilməsi, add-worker.yml playbook-da yeni node-un host adı qeyd edilməklə reallaşdırıla bilər:

  [root@master k8s-ansible]# ansible-playbook add-worker.yml
