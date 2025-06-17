# README - Passo a passo completo para corrigir erro NVIDIA-SMI no UmbrelOS (Debian 6.1) com driver NVIDIA 535

## Contexto

O problema original:

```
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
```

A GPU utilizada era uma NVIDIA GeForce GTX 750, e o sistema baseado em Debian 6.1 com o kernel 6.1.0-34 inicialmente, mas com o kernel 6.1.0-37 também instalado.

---

## Passo 1: Verificar o kernel em uso e os headers instalados

```bash
uname -r
```

> Resultado inicial: `6.1.0-34-amd64`

Verificar kernels instalados:

```bash
dpkg --list | grep linux-image
```

> Kernels listados:

- 6.1.0-34-amd64
- 6.1.0-37-amd64

Verificar se os headers do kernel em uso estão instalados:

```bash
ls /lib/modules/6.1.0-34-amd64/build
```

> Diretório ausente indicava headers faltando.

---

## Passo 2: Instalar headers do kernel correto

```bash
sudo apt install linux-headers-6.1.0-34-amd64
```

Para melhor compatibilidade futura, instalar também o kernel mais novo e seus headers:

```bash
sudo apt install linux-image-6.1.0-37-amd64 linux-headers-6.1.0-37-amd64
```

---

## Passo 3: Verificar e corrigir bootloader

Como o sistema usa systemd-boot e não GRUB, comandos como `grub-set-default` não funcionaram.

Verificações:

```bash
bootctl status
```

> systemd-boot não estava instalado.

---

## Passo 4: Reiniciar com kernel 6.1.0-37

Após instalar o kernel mais recente:

```bash
sudo reboot
```

Verificar kernel em uso:

```bash
uname -r
```

> Resultado esperado: `6.1.0-37-amd64`

Confirmar headers disponíveis:

```bash
ls /lib/modules/6.1.0-37-amd64/build
```

---

## Passo 5: Reinstalar e carregar drivers NVIDIA

Reinstalar drivers:

```bash
sudo apt install --reinstall nvidia-driver
```

Carregar módulos:

```bash
sudo modprobe nvidia
```

---

## Passo 6: Verificar com NVIDIA-SMI

```bash
nvidia-smi
```

> A GPU deve aparecer listada corretamente.

---

## Passo 7 (Opcional): Instalar CUDA Toolkit

```bash
sudo apt install nvidia-cuda-toolkit
nvcc --version
```

---

## Observações finais

- O erro foi causado pela falta dos headers do kernel em uso, impedindo que os módulos NVIDIA fossem construídos corretamente.
- A solução foi alternar para um kernel mais novo com headers presentes (6.1.0-37).
- Como o bootloader era systemd-boot (ou equivalente), comandos do GRUB não funcionaram.
- Após reinício e carregamento correto do kernel, os drivers funcionaram perfeitamente.

---

**Status final:** `nvidia-smi` reconhece a GPU e os módulos estão carregados corretamente.
