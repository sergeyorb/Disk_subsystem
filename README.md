# Работа с mdadm
<ol>
  <li>Используя vagrant Развернуть VM с 5 дисками</li>
  <li>Собрать Reid на выбор</li>
  <li>Создание конфигурационного файла mdadm.conf</li>
  <li>Сломать/починить raid</li>
  <li>Создать GPT раздел и 5 партиций</li>
</ol>

# 1.Используя vagrant Развернуть VM с 5 дисками
Начальный стенд взят от сюда  https://github.com/erlong15/otus-linux
В vagrantfile добавлены строки добавляющие пятый диск
<p>:sata5 => {<br>
                        <p>:dfile => './sata5.vdi',
                        <p>:size => 250, # Megabytes
                        <p>:port => 5
                        <p> },
                          
# 2.Собрать Reid на выбор
<ul>                          
<li>Проверяем блочные устройства командой:</li>
<p>lsscsi<br>
<li>Получаем следующий вывод</li>
<p>[vagrant@otuslinux ~]$ lsscsi<br>
<p>[0:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sda
<p>[3:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdb
<p>[4:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdc
<p>[5:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdd
<p>[6:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sde
<p>[7:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdf
  
<li>Создаём рейд следующей командой</li>  
<p>mdadm --create --verbose /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}<br>
 создали 5 рейд с 5 дисками (можно было с 4, но выдавал ошибку)
 
<li>Проверяем созданный рейд командой:</li>
cat /proc/mdstat 
<li>Полученный результат</li>
<p>[vagrant@otuslinux ~]$ cat /proc/mdstat<br>
<p>Personalities : [raid6] [raid5] [raid4]
<p>md0 : active raid5 sde[6] sdf[5] sdb[0] sdc[1] sdd[2]
      <p>1015808 blocks super 1.2 level 5, 512k chunk, algorithm 2 [5/5] [UUUUU]
<p>unused devices: <none>    
</ul>
  
# 3.Создание конфигурационного файла mdadm.conf
<ul>
<li>Создаём директорию mdadm</li>  
<li>Затем двумя командами создаём mdadm.conf</li>  
<p>echo "DEVICE partitions" > /etc/mdadm/mdadm.conf<br>
<p>mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf<br>  
</ul>  

# 4.Сломать/починить raid
<ul>
<li>Сломал raid командой:</li>
<p>mdadm /dev/md0 --fail /dev/sde<br>
<li>Проверяем Raid</li>
  
       <p>Number   Major   Minor   RaidDevice State
   
       <p>0       8       16        0      active sync   /dev/sdb
         
       <p>1       8       32        1      active sync   /dev/sdc
         
       <p>2       8       48        2      active sync   /dev/sdd
         
       <p>-       0        0        3      removed
         
       <p>5       8       80        4      active sync   /dev/sdf

       <p>6       8       64        -      faulty   /dev/sde 
         
Видим что третий диск статус removed
<li>Починил raid командой:</li>         
<p>mdadm /dev/md0 --add /dev/sde
  
Перед этим перезагрузил систему так как выходила ошибка о том, что устройство занято  
</ul>  

# 5.Создать GPT раздел и 5 партиций
<ul>
<li>Создаем раздел GPT на RAID</li>
<p>parted -s /dev/md0 mklabel gpt<br>  
<li>Создаем партиции</li>
<p>parted /dev/md0 mkpart primary ext4 0% 20%<br>
<p>parted /dev/md0 mkpart primary ext4 20% 40%<br>  
<p>parted /dev/md0 mkpart primary ext4 40% 60%<br>  
<p>parted /dev/md0 mkpart primary ext4 60% 80%<br>
<p>parted /dev/md0 mkpart primary ext4 80% 100%<br>  
<li>Монтируем по каталогам</li> 
<p>mkdir -p /raid/part{1,2,3,4,5}<br>
<p> for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done<br>  
</ul>  
