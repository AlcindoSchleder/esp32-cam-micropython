# esp32-cam-micropython

### Tutorial para preparar um ambiente de desenvolvimento com Toolchain e ESP-IDF para MicroPyhton

- Vamos construir o firmware micropython para a  [ESP32-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/)  uma placa de desnevolvimento que você pode encontrar aqui no Brasil na [FelipeFlop](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/) . Esta placa tem um sensor OV2640 sem qualquer chip (buffer) entre a ESP32 e a câmera. Existem muitas implementações para o Arduino ou diretamente usando Espressiif, mas eu queria usar MicroPython.
Assim, este tutorial é sobre compilar e gravar uma versão do MicroPython com suporte para i2c na  [ESP32-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/). Adicionalmente, eu incluí um projeto com um Web Server para tirar fotos e importá-las por streaming. Você também pode fazer o download de um firmware já compilado, atualizar sua placa ESP32 e tirar suas fotos rapidamente. mas este não é o nosso objetivo, apesar de eu deixar este [firmware](https://github.com/AlcindoSchleder/uPyCam/blob/master/firmware/esp32-cam-micropython.bin) que compilaremos aqui no repositório. O repositório da tsaarni tem uma wiki com instruções sobre como fazer isto, mas alguns dos tópicos não são facilmente compreendidos e eu tive alguns problemas com isto. Portanto eu tomei como base um segundo artigo e criei este aqui para que os makers amantes do python no Brasil possam desfrutar desta tecnologia.

## Hardware & Software

Na seguinte tabela, você encontrará o hardware e o software que você usará neste tutorial:

|  |  |
|--|--|
|ESP32-CAM |[![](https://uploads.filipeflop.com/2019/02/6WL90_02.jpg)]([https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/))|
|USB-TTL|[![](https://uploads.filipeflop.com/2017/07/4MD16-1.jpg)]([https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/))|
|esp32-camera-for-micropython|[![](https://avatars0.githubusercontent.com/u/85464?s=460&v=4)](https://github.com/tsaarni/esp32-camera-for-micropython)|
|micropython-with-esp32-cam|[![](https://lemariva.com/storage/temp/public/0c6/51f/27b/GitHub-Mark__400.png)](https://github.com/tsaarni/micropython-with-esp32-cam)|
|uPyCam|Este repositório|
|||

## DIY - Faça você mesmo

Eu fiz o upload do firmwarecompilado iseguinte repositório:  [AlcindoSchleder/uPyCam]([https://github.com/AlcindoSchleder/uPyCam](https://github.com/AlcindoSchleder/uPyCam)). Você pode autalizar o firmware com o seguinte comando:

```shell
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x1000 esp32-cam-micropython.bin
```

Entretanto, se você quer comilar o seu próprio MicroPython vamos colocar a mão na massa. Para iniciarmos nossos trabalhos, você vai precisar das seguintes dependências na dua máquina de desenvolvimento:

```shell
# linux fedora
sudo dnf install python3
sudo dnf install pip3

# linux Debian
sudo apt-get install python3-pip
pip3 install virtualenv
```

No Rwindows, siga este [artigo (inglês)](https://www.liquidweb.com/kb/install-pip-windows/)  para instalar o  `pip`  e o  `virtualenv`  com  `pip install virtualenv`.

Então coloque tudo que precisa dentro duma pasta:

```shell
mkdir micropython
cd micropython
export MICROPYTHON=$PWD
```
e inicie uma  máquina virtual - `python3 -m venv venv` - dentro desta pasta para instalar as dependências.
Para ativar a máquina virtual use o seguinte comando no linux:
```shell
# linux
. venv/bin/activate

# windows
myenv\Scripts\activate

pip3 install pyserial pyparsing
```
### Espressif IoT Development Framework (ESP-IDF)

A compilação do MicroPython para a placa  [ESP32-CAM](https://www.banggood.com/custlink/DKKGOWrqBj) é necessário configurar o Espressif IoT Development Framework (ESP-IDF). Digite o seguinte comando para realizarmos esta tarefa:
```shell
cd $MICROPYTHON
git clone https://github.com/espressif/esp-idf.git
```
Os próximos comando temos que encontrar o hash da versão correta desta placa. Para saber o hash abra o arquivo  [`micropython-with-esp32-cam/ports/esp32/Makefile`](https://github.com/tsaarni/micropython-with-esp32-cam/blob/esp32-camera-for-micropython/ports/esp32/Makefile) e lá encontraremos a seguinte definição na linha 30:
```
# the git hash of the currently supported ESP IDF version
ESPIDF_SUPHASH := 5c88c5996dbde6208e3bec05abc21ff6cd822d26
```
De posse do hash vamos executar os próximos comandos e então seguirmos em frente:
```shell
git checkout 5c88c5996dbde6208e3bec05abc21ff6cd822d26 
git submodule update --init
```
Outra forma de encontrar o hash é indo no diretório  `micropython-with-esp32-cam/ports/esp32`  e dar o comando  `make`  sem ter configurado o  ESP-IDF e você verá algo assim:
```shell
Use make V=1 or set BUILD_VERBOSE in your environment to increase build verbosity.

** WARNING **
The git hash of ESP IDF does not match the supported version
The build may complete and the firmware may work but it is not guaranteed
ESP IDF path:       /home/riva/esp/esp-idf
Current git hash:   3eecd43b31a765aae0231ff9a227786bac31b9a2
Supported git hash: 5c88c5996dbde6208e3bec05abc21ff6cd822d26
```
Alguns outros requisitos dependem do tipo do seu Sistema Operacional. Veja os  [pré requisitos](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html#get-started-get-prerequisites)  e instale na sua máquina.
Seguimos em frente, mais um comando para executar no terminal:
```shell
export ESPIDF=$MICROPYTHON/esp-idf
```

### Toolchain

Faça o download da Toolchain aqui:  [Toolchain Compilada](https://docs.espressif.com/projects/esp-idf/en/latest/get-started-legacy/index.html), São arquivos herdados de versões mais antigas porém ainda funcionam. Ou você pode compilar o ToolChain seguindo os passos abaixo. (Linux -  [requerimentos](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/linux-setup.html)):
**Obs:** *Os comandos abaixo na minha máquina, que é um I5, demorou cerca de 90 minutos o processo todo*
```shell
cd $MICROPYTHON
git clone https://github.com/espressif/crosstool-NG.git
cd crosstool-NG
git checkout esp32-2019r1
./bootstrap && ./configure --enable-local

./ct-ng xtensa-esp32-elf
./ct-ng build
chmod -R u+w builds/xtensa-esp32-elf
```

No windows você pode fazer o download do toolchain  [here](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/windows-setup-scratch.html). MacOS tem outros  [requerimentos](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/macos-setup-scratch.html).

Agora, no Linux/MacOS adicionamos o arquivos binários do toolchain para a variável de ambiente  `PATH`  assim:
```shell
export PATH=$PATH:$MICROPYTHON/crosstool-NG/builds/xtensa-esp32-elf/bin
```
Você pode inserir este comando dentro dos arquivos de inicialização do sheel do seu usuário. Pode ser feito no **<seu usuário>/.bashrc** ou no  **<seu usuário>/.bash_profile** dependendo da distribuição que você usa. Eu uso o o Fedora 29 e tem estes formatos acima.
Se você fez o download, extraia os arquivos e aponte o `$ PATH` para a pasta` bin` correta.

### OV2640 drivers

Ainda não temos os drivers que controlam a câmera OV2640 no ESP-IDF. Então faremos o seguinte:
```shell
cd $MICROPYTHON/esp-idf/components
git clone https://github.com/tsaarni/esp32-camera-for-micropython.git esp32-camera
```

### MicroPython for  [ESP32-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/)

Já adicionamos os drivers da nossa câmera OV2640 no ESP-IDF, vamos clonar a versão do MicroPython que suporta a câmera e compilar tudo usando as seguintes linhas:

```shell
cd $MICROPYTHON
git clone https://github.com/tsaarni/micropython-with-esp32-cam.git micropython
cd micropython/ports/esp32
git submodule update --init
make V=1 SDKCONFIG=boards/sdkconfig.esp32cam -j 
```

## Flash MicroPython on the board

A placa  [ESP32-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/)  não tem um chip CP2102 para conectá-lo diretamente com uma porta USB, usaremos, então, o nosso módulo conversor serial-USB para gravar o firmware e escrever nossos programas.

Conecte a USB UART do ESP32-CAM como descrita na tabela abaixo:
|**USB UART**|**ESP32-CAM**|
|--|--|
|TX|U0R|
|RX|U0T|
|DTR|IO0|
|5V|5V|
|GND|GND|
Aw você não tem o pino DTR disponível na sua placa (assim como a minha), você deve conectar o pino  `IO0`  para o pino  `GND`. E vamos gravar o nosso MicroPython. Para gravar programas MicroPyhton voê não precisa fazer o processo anterior.

Então conecte um cabo USB de 4 vias, (obs: você deve certificar-se que o cabo que está usando tem as 4 vias [RX, TX, 5V, GND], senão não irá conseguir fazer a transmissão para a sua placa) à uma das portas USB livres do seu PC e vamos em frente!!

```shell
cd $MICROPYTHON/micropython/ports/esp32
make deploy
```

O penúltimo comando montou o arquivo  `build/firmware.bin`  e este último faz o upload para a placa ESP32. Você deve ver algo como isto:

```shell
[...]
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```
dando continuidade, retire o jumper entre o pino  `IO0`  e o pino  `GND`  e reinicializa a placa pressionando o botão reset dela.

## Programming using MicroPython

Muitas pessoas preferem usar o uPyCraft para editar seus programas em python. eu particularmente não gostei de usar este programa, por isso optei em utilizar o Visual Studio Code (vscode) nesta tarefa.
Leia este artigo que ensina como utilizar o MicroPyhton com VSCode:  [#MicroPython: VSCode IntelliSense, Autocompletion & Linting capabilities](https://lemariva.com/blog/2019/08/micropython-vsc-ide-intellisense).

Use o seguinte código para conectar (atualize  `ssid_`  `wp2_pass`  com os valores certos da sua rede WiFi) ao  [ESP-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/):

```python
import network
ssid_ = <wlan-ssid>
wp2_pass = <wlan-password>

sta_if = network.WLAN(network.STA_IF)
sta_if.active(True)
sta_if.connect(ssid_, wp2_pass)
```

Então, instalamos os seguintes pacotes MicroPython digitando estas linhas dentro do terminal do VSCode:

```python
import upip
upip.install('picoweb')
upip.install('micropython-ulogging')
upip.install('ujson')
```

Clone o Projeto que está no meu repositério:  [uPyCam]([https://github.com/AlcindoSchleder/uPyCam](https://github.com/AlcindoSchleder/uPyCam))  e no sue PC configure o corretamente o arquivo  `boot.py`  com o SSID e a chave WPA2  do Wi-Fim então envie para a placa. Este projeto tira fotos com a câmera e mostra a imagem no seu browser a cada vez que você visitar o servidor Web que está ronando dentro da paca..

## Conclusões

Este tutorial pe sobre compilar uma versão para o Micropython que suporte a câmera OV2640. O firmware pode ser gravado na placa [ESP32-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/)  e tirar suas fotos. A resolução desta câmera não é o ideal, mas o baixo consumo de energia da ESP32 e a conectividade faz da [ESP32-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/)  o dispositivo ideal para segurança e usando aplicações remotas onde a fonte de energia é uma bateria. Futuramente, o MicroPython, assim como python, faz as atualizações do seu software facilmente. Obviamente que alguma performance é perdida, mas ainda assim, o resultado é aceitável.

## Tip and Errors

1.  Se você receber o seguinte erro ao compilar a toolchain (`./ct-ng build`):  `Build failed in step 'Extracting and patching toolchain components'`. Role a tela do terminal para cima e verifique qual arquivo está recebendo este erro. No meu caso eu recebi o seguinte:
    
    ```shell
    [EXTRA]    Extracting gcc-git-20df92b4
    [00:29] - 
    bzip2: Compressed file ends unexpectedly;
    [...]
    ```
    
    Para remover este erro faça o seguinte:
    
    ```shell
    cd crosstool-NG/.build/tarballs
    rm gcc-git-20df92b4.tar.bz2
    ```
    
    Execute novamente o passo de compilação do toolchain.
    
2.  O seguinte erro aparece porque a placa está rodando o firmware oficial e a versão do uasyncio não é compatível.
    
    ```python
    INFO:picoweb:30.000 <HTTPRequest object at 3ffc3590> <StreamWriter <socket>> "GET /"
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<string>", line 903, in <module>
    File "/lib/picoweb/__init__.py", line 298, in run
    File "/lib/uasyncio/core.py", line 161, in run_forever
    File "/lib/uasyncio/core.py", line 136, in run_forever
    File "/lib/uasyncio/__init__.py", line 60, in remove_writer
    TypeError: function takes 2 positional arguments but 3 were given
    ```
    
    Você tem que atualizar o arquivo  [`__init__.py`](https://github.com/micropython/micropython-lib/tree/master/uasyncio/uasyncio)  e  [`core.py`](https://github.com/micropython/micropython-lib/blob/master/uasyncio.core/uasyncio/core.py)  no  `lib/uasyncio`  por este do diretório  [`micropython-lib`](https://github.com/micropython/micropython-lib). Eu incluí estes arquivos no reposítório. Se você enviar o repositório completo, os arquivos serão sobrescritos e você não receberá este erro. Eu comprei minha primeira  placa [ESP32-CAM](https://www.filipeflop.com/produto/modulo-esp32-cam-com-camera-ov2640-2mp/)  da  [FilipeFlop](https://www.filipeflop.com).
    Entre no site da [FilipeFlop](https://www.filipeflop.com) e veja as promoções e toda a variedade de produtos que eles trabalham que é muito interessante.
