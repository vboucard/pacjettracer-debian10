# packettracer-debian10
Scripts pour installer PacketTracer sur Debian 10

# Sources
https://grupp-web.de/cms/2020/01/03/installation-of-packet-tracer-7-3-on-rpm-linux-systems-without-alien/

https://www.reddit.com/r/debian/comments/fbbrxg/is_anyone_able_to_install_packettracer_on_debian/

# Etapes
## Décompresser le DEB

	mkdir /tmp/PacketTracerInst
	cp PacketTracer_730_amd64.deb /tmp/PacketTracerInst
	cd /tmp/PacketTracerInst
	ar -xv PacketTracer_730_amd64.deb
	mkdir control
	tar -C control -Jxf control.tar.xz
	mkdir data
	tar -C data -Jxf data.tar.xz
	cd data

## Supprimer les anciennes installations

	sudo rm -rf /opt/pt
	sudo rm -rf /usr/share/applications/cisco-pt7.desktop
	sudo rm -rf /usr/share/applications/cisco-ptsa7.desktop
	sudo rm -rf /usr/share/icons/hicolor/48x48/apps/pt7.png

## Installation des fichiers

	sudo cp -r usr /
	sudo cp -r opt /

## Création de liens symboliques

	sudo ln -s /usr/lib64/libdouble-conversion.so.3.1.5 /usr/lib64/libdouble-conversion.so.1
	cd /usr/lib/x86_64-linux-gnu/
	sudo ln -s libicui18n.so.63 libicui18n.so.60
	sudo ln -s libicuuc.so.63 libicuuc.so.60

## Mise à jour du cache d'icônes et association

	sudo xdg-desktop-menu install /usr/share/applications/cisco-pt7.desktop
	sudo xdg-desktop-menu install /usr/share/applications/cisco-ptsa7.desktop
	sudo update-mime-database /usr/share/mime
	sudo gtk-update-icon-cache --force --ignore-theme-index /usr/share/icons/gnome
	sudo xdg-mime default cisco-ptsa7.desktop x-scheme-handler/pttp

## Lien symbolique vers l'exécutable

	sudo ln -sf /opt/pt/packettracer /usr/local/bin/packettracer

## Paramètres d'environnement dans /etc/profile

	PT7HOME=/opt/pt
	export PT7HOME
	QT_DEVICE_PIXEL_RATIO=auto
	export QT_DEVICE_PIXEL_RATIO

## libjpeg.so.8 à compiler

Si PacketTracer ne démarre pas, vérifier si il ne faut pas des librairies en plus :

	cd /opt/bin/bin
	ldd PacketTracer7 | grep 'not found'

Si il se plaint de ne pas trouver libjpeg.so.8, il va falloir la compiler :

	sudo apt install cmake nasm
	git clone https://github.com/libjpeg-turbo/libjpeg-turbo.git
	cd libjpeg-turbo
	cmake -G"Unix Makefiles" -DWITH_JPEG8=1
	make
	sudo mv ../libjpeg.so.8 /usr/lib/x86_64-linux-gnu/

## Si il ne démarre toujours pas

Copier le fichier /opt/bin/packettracer en /opt/bib/ptdebug et éditer ce dernier en enlevant les redirections, ce qui permettra de voir
où ça bloque

	sudo cp -p /opt/pt/packettracer /opt/pt/ptdebug
	sudo sed -i "s/>.*$//g" /opt/pt/ptdebug
