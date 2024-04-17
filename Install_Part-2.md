Siguiendo con la instalacion de Arch Linux, luego de reiniciar, nos logueamos y proseugimos.

## Drivers de video

Si la tarjeta grafica es amd, hay que instalar el paquete `xf86-video-amdgpu`, para el caso de intel es `xf86-video-intel` y para nvidia es `nvidia` y `nvidia-utils`. Si fuera una maquina virtual, hay que instalar `xf86-video-qxl`

## Instalando paquetes para configurar el entorno

El Display Server sera Xorg, `sudo pacman -Syu xorg xorg-server xorg-xinit`

El Display Manager, sera lightdm: `sudo pacman -Syu lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings`

El Desktop Environment, sera qtile: `sudo pacman -Syu qtile`


## Instalando mas paquetes que seran necesarios

`sudo pacman -Syu mpd mpc nitrogen pipewire pipewire-alsa pipewire-pulse wireplumber pavucontrol zsh zsh-syntax-highlighting zsh-autosuggestions wget curl p7zip unzip lsd bat kitty thunar chromium firefox picom lxappearance flameshot rofi arandr python-pip python-pipenv papirus-icon-theme`

- `mpd` y `mpc` para la musica. 
- `nitrogen` para los wallpapers. 
- `pipewire`, `pipewire-alsa`, `pipewire-pulse`, `wireplumber` para servidor de audio y administrar el audio bluetooth
- `pavucontrol` para administrar los sonidos del sistema mediante GUI 
- `zsh zsh-syntax-highlighting zsh-autosuggestions` para zsh y los plugins
- `wget`, `curl`, `p7zip`, `unzip`, `lsd`, `bat` para ir teniendo
- `kitty` como la shell
- `thunar` como explorador de archivos
- `chromium firefox` como navegadores (aunque luego isntalaremos brave)
- `picom` opcional. Se usa para manejar las transparencias
- `lxappearance` es para poder administrar los temas de GTK+. Algunas aplicaciones podrian necesitarlo, asi que lo instalamos por las dudas
- `flameshot` para hacer capturas de pantalla
- `rofi` para usar como menu de apps
- `arandr` para configurar la resolucion de pantalla mediante gui
- `python-pip` y `python-pipenv` para instalar paquetes de python
- `papirus-icon-theme` para tener un tema de iconos

## Habilitamos los servicios necesarios

- `systemctl --user enable mpd`
- `sudo systemctl enable pipewire-pulse` (puede que falle, pero no importa)
- `sudo systemctl enable lightdm`

Creamos en el escritorio un archivo `.xprofile`. Ester archivo sera lo primero que lea qtile (o lightdm, no se bien) para cargar los servicios necesarios:
```
xrandr --output HDMI-A-0 --mode 1024x768 --pos 1280x0 --rotate normal --output DVI-D-0 --off --output DisplayPort-0 --primary --mode 1280x720 --pos 0x0 --rotate normal

nitrogen --restore &

flameshot &
```

## Instalando las fuentes necesarias (y quizas otras no tanto)

Las fuentes que vamos a instalar son:
- `ttf-ms-fonts`
- `nerd-fonts-jetbrains-mono` 
- `ttf-font-awesome`
- `ttf-font-awesome-4`
- `ttf-material-design-icons`
- `Iosevka`
- `Hack Nerd Fonts`
- `icommon`

Para las `ttf-ms-fonts`:
1. `mkdir ~/Downloads` 
2. `git clone https://aur.archlinux.org/ttf-ms-fonts.git`
3. Moverse a la carpeta descargada
4. Ejecutar `makepkg -si`

Para las `nerd-fonts-jetbrains-mono`:
1. `cd ~/Downloads`
2. `git clone https://aur.archlinux.org/nerd-fonts-jetbrains-mono.git`
3. Moverse a la carpeta descargada.
4. Ejecutar `makepkg -si`

Para las `ttf-font-awesome`:
1. `sudo pacman -S ttf-font-awesome`

Para las `ttf-font-awesome-4`:
1. `cd ~/Downloads`
2. `git clone https://aur.archlinux.org/ttf-font-awesome-4.git`
3. Moverse a la carpeta descargada.
4. Ejecutar `makepkg -si`

Para las `ttf-material-design-icons`:
1. `cd ~/Downloads`
2. `git clone https://aur.archlinux.org/ttf-material-design-icons.git`
3. Moverse a la carpeta descargada.
4. Ejecutar `makepkg -si`

# las hack nerd son claves para algunos emojis/simbolos
Para las `Hack Nerd Fonts`:
1. `sudo su`
2. `cd /usr/share/fonts`
3. `wget https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/Hack.zip`
4. `unzip Hack.zip`
5. `rm Hack.zip`

Para las `Iosevka`:
1. Asegurarse de no estar como root
2. cd ~/Downloads
3. `wget https://fontlot.com/downfile/5baeb08d06494fc84dbe36210f6f0ad5.105610`
4. `mv 5baeb08d06494fc84dbe36210f6f0ad5.105610 Iosevka.zip`
5. `unzip Iosevka.zip`
6. Descomprimira 2 carpetas. Buscar en ambas todos los archivos que sean `.ttf` y moverlos a `/usr/share/fonts/`

Para las `icommon`:
1. `cd ~/Downloads`
2. `wget 'https://www.dropbox.com/s/hrkub2yo9iapljz/icommon.zip'`
4. `unzip icommon.zip`
5. `sudo cp icomoon/* /usr/share/fonts/`


Otras que vamos a instalar son:
- ttf-dejavu
- noto-fonts-emoji

`sudo pacman -Syu ttf-dejavu noto-fonts-emoji`


## Instalamos algunos paquetes desde el aur

Instalamos el navegador Brave:
1. `cd ~/Downloads`
2. `git clone https://aur.archlinux.org/brave-bin.git`
3. Nos movemos a la carpeta descargada.
4. `makepkg -si`

## Copiamos archivos de configuracion

En la carpeta Resources, estan las carpeta de rofi, kitty, qtile y las que sean necesarias que van dentro de la carpeta `~/.config`. Por lo que las copiamos y las pegamos donde corresponde.

Finalmente reiniciamos y rogamos que todo salga bien. Al reiniciar deberia atendernos lightdm, y cuando nos logueemos, qtile deberia verse bonito.

## Configuramos la ZSH 

Instalamos el ultimo plugin que es zsh-sudo. Para ello:
1. Nos movemos a la carpeta donde esten los demas plugins `cd /usr/share/zsh/plugins`
2. `sudo mkdir zsh-sudo && cd zsh-sudo`
3. `sudo wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/plugins/sudo/sudo.plugin.zsh`


Ahora instalaremos la powerlevel10k, para ello ejecutamos:
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

si no se inicia el configurador de la powerlevel, salimos y volvermos a abrir la terminal, eso deberia hacer que nos aparezca la opcion para configurar. 

Si se quiere, en la carpeta Resources hay un modelo de `.zshrc`. Tomar de ahi lo que se quiera. Estan las configuraciones para las teclas de `control`, `home`, `end`, etc.
Tambien esta la configuracion de los plugins y los aliases. Asegurarse de que las rutas que se pongan, sean las adaptadas a la del sistema actual.

Para configurar zsh por defecto ejecutamos: `chsh -s /bin/zsh`

Reiniciamos y quedaria casi listo. Queda configurar mpd y mpc para la musiquita, nitrogen para seleccionar un wallpaper, como se administra el bluetooth, y algun otro detalle.

instale gvfs para manejar sistemas de archivos virtuales, esto incluye a la papelera y cuando se conecta unc elular
