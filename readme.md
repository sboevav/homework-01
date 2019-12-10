# Решение основной части ДЗ

Как я уже описал раньше в третьем ДЗ, это задание пытался отработать на виртуалке под win 10 и win 7. Но после пары недель мучений это так и не удалось. Возможно, это особенности виртуалки под windows, а может дело в каких-то настройках, которые еще пока не освоил. После перехода на ubuntu это задание отработало сразу.  
Основное задание сделано без описания выполняемых действий, т.к. не ожидал что сразу получится :-). Но, собственно, просто выполнены последовательно все команды из методички, без всяких изменений. В результате получен образ, создан Vagrantfile и протестирована его загрузка в каталоге test.  

Публикация образа в Vagrant Cloud:  
		aleksei@linux:~/linux/homework-01/packer$ vagrant cloud publish --release sboevav/centos-7-5 1.0 virtualbox centos-7.7.1908-kernel-5-x86_64-Minimal.box  
	You are about to publish a box on Vagrant Cloud with the following options:  
	sboevav/centos-7-5:   (v1.0) for provider 'virtualbox'  
	Automatic Release:     true  
	Do you wish to continue? [y/N] y  
	==> cloud: Creating a box entry...  
	==> cloud: Creating a version entry...  
	==> cloud: Creating a provider entry...  
	==> cloud: Uploading provider with file /home/aleksei/linux/homework-01/packer/centos-7.7.1908-kernel-5-x86_64-Minimal.box  
	==> cloud: Releasing box...  
	Complete! Published sboevav/centos-7-5  
	tag:             sboevav/centos-7-5  
	username:        sboevav  
	name:            centos-7-5  
	private:         false  
	downloads:       0  
	created_at:      2019-12-10T22:51:29.083+04:00  
	updated_at:      2019-12-10T22:52:52.424+04:00  
	current_version: 1.0  
	providers:       virtualbox  
	old_versions:    ...  

# Решение ДЗ сборки ядра из исходников 

1. Посмотрим текущую версия ядра  
		[vagrant@kernel-update ~]$ uname -r  
	3.10.0-957.12.2.el7.x86_64

2. Установим wget  
		[vagrant@kernel-update ~]$ sudo yum install wget  
	...  

	Installed:  
	  wget.x86_64 0:1.14-18.el7_6.1                                                 

	Complete!  

3. Копируем исходные файлы версии 5.4  
		[root@kernel-update vagrant]# wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.4.tar.xz  
	--2019-12-06 14:15:04--  https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.4.tar.xz  
	Resolving cdn.kernel.org (cdn.kernel.org)... 151.101.85.176, 2a04:4e42:14::432  
	Connecting to cdn.kernel.org (cdn.kernel.org)|151.101.85.176|:443... connected.  
	HTTP request sent, awaiting response... 200 OK  
	Length: 109441440 (104M) [application/x-xz]  
	Saving to: ‘linux-5.4.tar.xz’  
	...  
	100%[======================================>] 109,441,440  491KB/s   in 3m 27s   

	2019-12-06 10:22:59 (516 KB/s) - 'linux-5.4.tar.xz' saved [109441440/109441440]  

4. Распаковываем скачанный архив  
		[root@kernel-update vagrant]# tar -xvf linux-5.4.tar.xz -C /usr/src  

5. Переходим в каталог, куда распаковали исходники ядра  
		[root@kernel-update src]# cd /usr/src/linux-5.4/  

6. Устанавливаем инструменты для компиляции пакетов  
		[root@kernel-update src]# yum groupinstall "Development Tools"  
	...  

	Dependency Updated:  
	  elfutils-libelf.x86_64 0:0.176-2.el7     elfutils-libs.x86_64 0:0.176-2.el7   
	  glibc.x86_64 0:2.17-292.el7              glibc-common.x86_64 0:2.17-292.el7   
	  libgcc.x86_64 0:4.8.5-39.el7             libgomp.x86_64 0:4.8.5-39.el7        
	  libstdc++.x86_64 0:4.8.5-39.el7          rpm.x86_64 0:4.11.3-40.el7           
	  rpm-build-libs.x86_64 0:4.11.3-40.el7    rpm-libs.x86_64 0:4.11.3-40.el7      
	  rpm-python.x86_64 0:4.11.3-40.el7       

	Complete!  

7. Устанавливаем инструменты для компиляции пакетов  
		[root@kernel-update src]# yum install ncurses-devel openssl-devel bc  
	...  
	Dependency Updated:  
	  e2fsprogs.x86_64 0:1.42.9-16.el7       e2fsprogs-libs.x86_64 0:1.42.9-16.el7  
	  krb5-libs.x86_64 0:1.15.1-37.el7_7.2   libcom_err.x86_64 0:1.42.9-16.el7      
	  libss.x86_64 0:1.42.9-16.el7           openssl.x86_64 1:1.0.2k-19.el7         
	  openssl-libs.x86_64 1:1.0.2k-19.el7   

	Complete!  

8. Создаем свою конфигурацию для ядра  
		[root@kernel-update src]# make menuconfig  
	...  
	*** End of the configuration.  
	*** Execute 'make' to start the build or try 'make help'.  

9. Доустанавливаем необходимые модули для компиляции (эти модули потребовал make при первом запуске)  
		[root@kernel-update linux-5.4]# yum install elfutils-libelf-devel  
	...  

	Installed:  
	  elfutils-libelf-devel.x86_64 0:0.176-2.el7                                    

	Complete!  

10. Запускаем компиляцию ядра (Здесь приведен вывод повторного запуска команды компиляции, т.к. после первого запуска не дождался завершения работы, а затем увидел сообщение об аварийном завершении работы виртуалки, поэтому перезапустил компиляцию)  
		[root@kernel-update linux-5.4]# make -j 4  
	  DESCEND  objtool  
	  CALL    scripts/atomic/check-atomics.sh  
	  CALL    scripts/checksyscalls.sh  
	  CHK     include/generated/compile.h  
	  TEST    posttest  
	  Building modules, stage 2.  
	  MODPOST 2448 modules  
	arch/x86/tools/insn_decoder_test: success: Decoded and checked 5902031 instructions  
	  TEST    posttest  
	arch/x86/tools/insn_sanity: Success: decoded and checked 1000000 random instructions with 0 errors (seed:0x3fa9de5a)  
	Kernel: arch/x86/boot/bzImage is ready  (#1)  

11. Устанавливаем ядро и модули  
		[root@kernel-update linux-5.4]# make modules_install install  
	...  
	  DEPMOD  5.4.0  
	sh ./arch/x86/boot/install.sh 5.4.0 arch/x86/boot/bzImage \
		System.map "/boot"  

12. Чтобы система грузилась с использованием нового ядра, внесем изменения в grub - установим значение GRUB_DEFAULT=0. Данной настройкой мы говорим загрузчику использовать первое ядро для загрузки (первым идет последнее по версии ядро)  
		[root@kernel-update linux-5.4]# vi /etc/default/grub  
		[root@kernel-update linux-5.4]# cat /etc/default/grub  
	GRUB_TIMEOUT=1  
	GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"  
	GRUB_DEFAULT=0  
	GRUB_DISABLE_SUBMENU=true  
	GRUB_TERMINAL_OUTPUT="console"  
	GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto"  
	GRUB_DISABLE_RECOVERY="true"  

13. Применяем настройки для grub:  
		[root@kernel-update linux-5.4]# grub2-mkconfig -o /boot/grub2/grub.cfg  
	Generating grub configuration file ...  
	Found linux image: /boot/vmlinuz-5.4.0  
	Found initrd image: /boot/initramfs-5.4.0.img  
	Found linux image: /boot/vmlinuz-3.10.0-957.12.2.el7.x86_64  
	Found initrd image: /boot/initramfs-3.10.0-957.12.2.el7.x86_64.img  
	done  

14. Перезагружаем виртуалку, после этого проверяем версию ядра  
		[vagrant@kernel-update ~]$ uname -r  
	5.4.0  

15. Перезагружаем виртуалку еще раз и выбираем в меню загрузки систему со старой версией ядра, загружаемся и проверяем версию ядра  
		[vagrant@kernel-update ~]$ uname -r  
	3.10.0-957.12.2.el7.x86_64  



