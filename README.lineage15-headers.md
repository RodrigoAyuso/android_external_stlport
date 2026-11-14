# Headers STLport cm-14.1 com Bionic do Oreo

Patch incremental para o checkout com fontes na revisao
984b843229e8993571b5d10dad05ec9eae27e129 de android_external_stlport.
O alvo libstlport agora e encontrado; a falha enviada acontece durante
a compilacao C++ dos streams, antes do link.

## Ajustes

- _cstdio.h deixa de importar gets no Android, onde a declaracao nao esta
  disponivel nos headers usados pela build. Isso remove uma declaracao
  using; nao adiciona implementacao da funcao nem troca o runtime C++.
- _cwchar.h usa stl/_cstddef.h no Android. Esse header do proprio STLport
  inclui stddef.h e importa size_t/ptrdiff_t conforme sua configuracao.
  O codigo antigo tentava libstdc++/include/cstddef, removido da arvore
  Bionic 15.1. O header nativo new continua disponivel e nao e alterado.

Os caminhos para plataformas nao Android sao preservados. Nao ha mudanca
nos layouts de containers ou substituicao de STLport por libc++.

## Aplicar no CachyOS

Salve o patch na raiz Android:

```sh
cd /home/rodrigoayuso/Android/LineageOS-15.1
git apply --check lineage15_fix_stlport_oreo_headers.patch
git apply lineage15_fix_stlport_oreo_headers.patch
```

## Compilar no Distrobox com o ambiente Python 2 existente

```sh
m -j4 libstlport
```

Se esse alvo terminar com sucesso:

```sh
m -j4 gralloc.lge_msm7x27a lge-gpu-probe
```

A alteracao de headers e detectada pelas dependencias da compilacao.
Envie o proximo erro completo, caso ainda haja incompatibilidades.
Gere o ZIP e rode o verificador de pacotes depois que os modulos compilarem.
Mantenha o backend software durante esta etapa de recuperacao.

## Validacao realizada

Foram executados 14 casos de compilacao com os headers STLport reais,
configuracao minima de suas macros e headers/runtime C do host. Os testes
reproduzem as falhas originais de gets e cstddef, compilam e executam os
casos corrigidos com __ANDROID__ ou ANDROID, verificam os aliases de tipos
e preservam o caminho de headers nativos para plataformas nao Android.

Esses testes isolam os erros de headers mostrados no log. Nao representam
a compilacao ARM completa de libstlport nem validacao da ABI dos blobs GPU.
O build completo precisa ser repetido no ambiente do usuario.

Fontes: [STLport usado](https://github.com/LineageOS/android_external_stlport/tree/984b843229e8993571b5d10dad05ec9eae27e129/stlport/stl) e
[headers C++ minimos da Bionic 15.1](https://github.com/LineageOS/android_bionic/tree/lineage-15.1/libstdc%2B%2B/include).
