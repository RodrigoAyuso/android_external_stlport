# STLport: typeinfo ausente na Bionic do Oreo

Patch incremental sobre as fontes cm-14.1 e o patch anterior de gets/cstddef.
O novo log confirma que esses dois erros foram superados; messages.cpp e
locale_impl.cpp agora falham em libstdc++/include/typeinfo.

A configuracao Android ja define _STLP_NO_RTTI, _STLP_NO_EXCEPTIONS e
_STLP_NO_EXCEPTION_HEADER. Este patch tambem define _STLP_NO_TYPEINFO,
selecionando o caminho existente em stl/_typeinfo.h e stl/_exception.h.
Nao copia headers de outra STL nem adiciona um novo runtime C++.

## Aplicacao no CachyOS

Mantenha o patch anterior aplicado e salve este na raiz da arvore Android:

```sh
cd /home/rodrigoayuso/Android/LineageOS-15.1
git apply --check lineage15_fix_stlport_typeinfo.patch
git apply lineage15_fix_stlport_typeinfo.patch
```

## Compilacao manual no Distrobox

Com o ambiente e lunch lineage_vee3-userdebug carregados:

```sh
m -j4 libstlport
```

Se terminar sem erros:

```sh
m -j4 gralloc.lge_msm7x27a lge-gpu-probe
```

Nao precisa apagar out. Mantenha o backend software durante a recuperacao,
ate confirmar a compilacao e instalacao de todas as dependencias da GPU.
Os avisos de unused parameter, register e tautological-compare enviados
nao sao o motivo da interrupcao mostrada neste log.

## Validacao e limites

Usados os arquivos reais de configuracao Android, o wrapper publico typeinfo,
os headers internos typeinfo/exception e suas dependencias de configuracao
na revisao 984b843229e8993571b5d10dad05ec9eae27e129 do STLport.

Foram verificados 16 casos de sintaxe: original/corrigido, C++98/C++14,
host 32/64 bits e perfil biblioteca/consumidor, com RTTI e exceptions
desativados e headers C++ do host bloqueados por -nostdinc++.
Os originais falham no header nativo ausente; os corrigidos compilam.
Dois casos corrigidos tambem foram ligados e executados no host.

Isso valida a correcao do include e a declaracao de bad_cast derivada de
exception no caminho de fallback. Nao constitui build ARM completo, teste
da ABI dos blobs Adreno ou validacao de boot/tela. Repita a compilacao no
Distrobox antes de gerar e instalar uma nova ROM.

Fontes:
- https://github.com/LineageOS/android_external_stlport/blob/984b843229e8993571b5d10dad05ec9eae27e129/stlport/stl/config/_android.h
- https://github.com/LineageOS/android_external_stlport/blob/984b843229e8993571b5d10dad05ec9eae27e129/stlport/stl/_typeinfo.h
