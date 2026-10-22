# Tema Beamer — IFCE Baturité (template de lista de exercícios)

Tema LaTeX/Beamer com três tipos de slide — **título**, **questão** e,
futuramente, **gabarito** — para listas de exercícios do IFCE Campus
Baturité, pensado para dupla exibição (projeção e leitura em celular).

Este repositório é **só o template**: os placeholders ficam genéricos
(`<tipo de conteúdo>`, `<nome da disciplina>`, etc.) até serem
preenchidos numa lista de exercícios real. Nenhuma lista real entra
aqui — cada lista é feita num **clone local separado** deste
repositório (veja "Fluxo de trabalho" abaixo).

## Estrutura

```
.
├── beamerthemeIFCEBaturite.sty   # o tema
├── main.tex                       # exemplo/ponto de partida (capa + 4 questões + gabarito, tudo genérico)
├── main.pdf                       # PDF já compilado do exemplo, pra conferir sem compilar
├── .gitignore                     # ignora lixo de compilação (.aux, .log, .nav, ...)
└── README.md
```

Não há pasta `assets/` aqui: nenhuma logomarca é versionada neste
repositório, nem a institucional, nem a de curso (veja "Ativos
institucionais" abaixo). Isso mantém o template leve e evita
duplicar binários a cada clone.

## Como compilar

```bash
xelatex main.tex
xelatex main.tex   # 2ª passada, fecha numeração/referências
```

Requer **XeLaTeX** ou **LuaLaTeX** (o tema usa `fontspec` para Carlito
e Open Sans). A fonte Carlito já vem em muitas distribuições TeX Live;
se faltar, baixe em fonts.google.com. Se "Open Sans" não estiver
instalada, o tema cai para Carlito automaticamente, sem travar a
compilação.

## Tamanho de página: por que é diferente do Beamer padrão

O Beamer **não** aceita `paperwidth=`/`paperheight=` como opção de
`\documentclass` (só entende `aspectratio=`, que fixa a altura em uns
9,6cm). Como este tema usa tamanhos de fonte **absolutos** (24pt, 15pt,
etc.) pensados para a escala física do PPTX original — não frações
relativas —, ele precisa que o slide tenha o tamanho físico real do
PPTX: **33,87 × 19,05 cm** (16:9, arredondamento oficial informado
para o tamanho de slide). O tema força isso sozinho via o pacote
`geometry`, então basta:

```latex
\documentclass{beamer}
\usetheme{IFCEBaturite}
```

sem nenhuma opção de papel no `\documentclass`.

## Comandos disponíveis

### `config.tex` — as 4 imagens + os metadados desta lista

Tudo que configura a lista (nada de conteúdo) fica num arquivo só,
`config.tex`, que o `main.tex` carrega com `\input{config.tex}`. As
4 imagens ficam juntas, no topo, cada uma numa linha:

```latex
\IFCEativosInstitucionais
  {\includegraphics[width=7.25cm]{/caminho/logo-ifce-baturite.pdf}}  % 1. logo da instituição
  {\includegraphics[width=0.37cm]{/caminho/icone-ifce-mini.pdf}}     % 2. ícone minúsculo (rodapé)

\def\ImagemCapa{\includegraphics[width=17.87cm]{/caminho/imagem-de-capa.pdf}}  % 3. imagem de capa

\IFCEmetadados
  {IFCE Baturité}                                      % instituição
  {<curso/turma>}
  {<nome da disciplina>}
  {<tipo de conteúdo>}
  {<descrição do conteúdo>}
  {<assunto/tópico>}
  {Prof. <nome do professor>}                         % inclua Prof./Profa.
  {\includegraphics[width=2.0cm]{/caminho/logo-do-curso.pdf}}        % 4. logo do curso
```

No template (sem nenhuma imagem real ainda), os 4 campos vêm como
`\IFCEplaceholder{...}` — troque cada um por `\includegraphics[width=...]{caminho}`
quando tiver a arte. O caminho pode ser relativo a `config.tex` (ex.
`fig/logo.pdf`) ou absoluto.

A logomarca institucional (1) e o ícone (2) são **sempre os mesmos**
em qualquer lista — os arquivos não vêm com o template, ficam num
diretório seu, fora do controle de versão (veja "Fluxo de trabalho").
A imagem de capa (3) e a logo do curso (4) mudam a cada lista.

Os 7 primeiros campos de `\IFCEmetadados` alimentam **automaticamente**
o slide de título e o cabeçalho/rodapé de todo slide de questão — não
são repetidos em cada `\question`. Os campos `<disciplina>`, `<tipo
de conteúdo>` e `<assunto>` aparecem separados pelo símbolo "▪"
(`\IFCEsep`, desenhado com `\rule` — não depende do glifo existir na
fonte), tanto na faixa superior do slide de questão quanto, com
instituição/curso/professor, no rodapé.

### Slide de título — `\CapaIFCE{conteúdo à direita}`

```latex
\CapaIFCE{\ImagemCapa}
```

Só recebe a ilustração da capa (normalmente `\ImagemCapa`, definida em
`config.tex`); todo o resto vem dos comandos de `config.tex` acima.

### Slide de questão — duas formas

#### Uso padrão: `\question{título/edição}{conteúdo}`

```latex
\question
  {<edição/informação da questão>}   % ex.: "ENEM 2023", "UECE 2022.1"
  {<enunciado da questão, com \begin{alternativas}...\end{alternativas} no fim>}
```

É um **texto corrido**: o conteúdo enche primeiro a coluna esquerda,
continua na direita e, se ainda sobrar, abre sozinho outro(s)
slide(s) seguinte(s) — sem precisar de nenhum comando extra. Nos
slides de continuação, o crachá repete o mesmo número e o título
ganha `[continuação]`; o slide anterior mostra `[continua >]` no
canto superior direito, na mesma linha de base do título. O corpo
fica **sempre em 24pt** — a fonte nunca é reduzida para caber; quem
decide onde corta é o próprio mecanismo (`\vsplit` do TeX, medindo a
altura real do texto composto), nunca o autor da lista.

O texto é **justificado** (o `\question` anula o `\raggedright` que o
Beamer aplica por padrão) e todo **parágrafo novo** recebe recuo de
**2,0cm** — em qualquer coluna ou slide; um parágrafo que apenas
*continua* de uma coluna/slide para o seguinte não recebe recuo.

Para forçar manualmente a passagem para a próxima coluna/slide antes
do ponto onde o texto estouraria sozinho (por exemplo, para manter
uma figura e seu `\caption` juntos, sem o texto ficando cortado no
meio), use:

```latex
\vfill\penalty-10000
```

em qualquer ponto do conteúdo.

#### Uso manual: `\question*{título/edição}{coluna esquerda}{coluna direita}`

```latex
\question*
  {<edição/informação da questão>}
  {<conteúdo da coluna esquerda>}
  {<conteúdo da coluna direita>}
```

As duas caixas são **independentes** — nada passa de uma coluna para
a outra, e não há corte automático. Use quando quiser decidir
exatamente o que vai em cada coluna (ex.: enunciado à esquerda,
figura grande à direita). Como não há `\vsplit` aqui, é
responsabilidade de quem escreve a lista garantir que o conteúdo de
cada coluna caiba na altura disponível (~14,6cm) — o TikZ não avisa
se o texto ultrapassar o rodapé, então vale a pena conferir
visualmente o PDF gerado. Vale a mesma regra de recuo do uso manual
original: 2,0cm na primeira linha dos parágrafos da coluna esquerda;
coluna direita sem recuo.

Em ambas as formas, o número do crachá avança sozinho a cada questão
**nova** (`"01"`, `"02"`, ...) — continuações da mesma questão
repetem o número — e sua linha de base é sempre igual à do título ao
lado (dois nós do TikZ ancorados na mesma coordenada de baseline). O
cabeçalho (`<disciplina> ▪ <tipo de conteúdo> ▪ <assunto>`) e o
rodapé (`<instituição> ▪ <curso> ▪ <professor>` à esquerda, `Slide X
de N` à direita) são montados automaticamente a partir dos metadados
globais, em ambas as formas.

### Alternativas — `alternativas`, `\itemCorreto`, `\itemCorreto*`

```latex
\begin{alternativas}
  \item primeira alternativa.
  \itemCorreto segunda alternativa.   % marca esta como a resposta correta
  \item terceira alternativa.
\end{alternativas}
```

Ambiente de lista com rótulos `a)`, `b)`, `c)`, ... Cada alternativa
nunca é cortada ao meio por uma quebra de coluna/slide, e a primeira
alternativa sempre fica junto do início da questão (nunca sozinha,
"órfã", no topo da coluna/slide seguinte).

- `\itemCorreto` marca a alternativa correta da questão que está
  sendo montada — essa letra vai para o `\gabarito` automaticamente.
- `\itemCorreto*` (com asterisco) marca uma alternativa correta
  **sem gabarito oficial disponível** — aparece no `\gabarito` com um
  `*` ao lado da letra, e uma nota de rodapé é adicionada
  automaticamente ao(s) slide(s) de gabarito avisando disso.

Uma questão sem nenhum `\itemCorreto`/`\itemCorreto*` aparece no
gabarito com `?` no lugar da letra.

### Gabarito — `\gabarito`

```latex
\gabarito   % uma vez, no fim do documento
```

Monta sozinho o(s) slide(s) de gabarito, no formato "número – letra",
na ordem em que as questões aparecem no documento (mudou a ordem das
questões? Basta recompilar — o gabarito acompanha). Cabem até 70
respostas por slide (7 colunas × 10 linhas); se a lista tiver mais
questões que isso, o `\gabarito` abre quantos slides forem
necessários sozinho. O crachá desses slides mostra `GAB` em vez de um
número.

**Exige duas compilações seguidas** (`xelatex main.tex` duas vezes):
na primeira passada o LaTeX ainda não sabe quantas questões existem
nem quais são as respostas; na segunda, já sabe e monta o gabarito
corretamente.

### Legendas de figura/tabela (`\caption`)

Figuras e tabelas usam os ambientes nativos do LaTeX (`figure`,
`table`) normalmente, com a fonte/crédito no próprio `\caption`:

```latex
\begin{figure}
\includegraphics[width=10cm]{fig/arquivo.pdf}
\caption{SOBRENOME, N. Disponível em: www.site.com.br. Acesso em:
  1 jan. 2020 (adaptado).}
\end{figure}
```

O tema ajusta a aparência da legenda: sem o rótulo `Figure:` na
frente, alinhada à direita, em 11pt.

Para um crédito de texto **sem** figura (ex.: fonte de uma citação),
use:

```latex
\par{\raggedleft\fontsize{15}{18}\selectfont Fonte: ...\par}
```

(o `\par` inicial é obrigatório — sem ele, o `\raggedleft` vaza para
o parágrafo anterior).

### `\IFCEplaceholder[largura]{legenda}`

Caixa tracejada verde para marcar onde entra uma figura, antes de
inserir a arte final de uma questão específica. Respeita a largura
informada (o texto quebra linha dentro dela) e nunca "vaza" para
fora da caixa. A altura é sempre 0,55× a largura — para uma imagem
larga/achatada (como a logomarca institucional), use uma largura de
demonstração menor do que a largura real, só para caber no espaço
disponível (veja o comentário no `main.tex`).

### `\IFCEimagemDummy{largura}{altura}`

"Foto" genérica desenhada em TikZ (moldura + sol + montanhas) — só
para preencher os placeholders de imagem do `main.tex` de
demonstração e dar uma ideia mais realista do resultado final. Não é
pensada para uso numa lista de exercícios real.

## Tipografia (tamanhos absolutos)

| Elemento                                             | Fonte      | Tamanho | Múltiplo | Entrelinha |
|-------------------------------------------------------|------------|---------|----------|------------|
| Nome do curso (capa, ao lado da logo)                  | Open Sans  | 16pt    | 1,0      | 19,2pt     |
| Demais textos de identificação (capa)                  | Carlito    | 24pt    | 1,15     | 33,12pt    |
| Cabeçalho/rodapé das faixas (questão)                  | Carlito    | 15pt    | 1,0      | 18pt       |
| Número da questão (dentro do crachá)                   | Carlito    | 40pt    | 0,9      | 43,2pt     |
| Título/edição da questão (ao lado do crachá)            | Carlito    | 20pt    | 1,15     | 27,6pt     |
| Corpo da questão (colunas de conteúdo)                  | Carlito    | 24pt    | 1,15     | 33,12pt    |

"Múltiplo X" foi convertido para entrelinha do LaTeX pela convenção
`1,2 × tamanho × múltiplo` (equivalente ao espaçamento "simples" do
PowerPoint/Word). Se algum texto parecer com entrelinha maior/menor do
que o esperado, esse é o fator a ajustar — procure por
`\IFCEfont...` no início do `.sty`.

## Cores

| Elemento                              | Valor      |
|----------------------------------------|------------|
| Verde institucional (textos, crachá)    | `#37A033`  |
| Verde claro (faixas de fundo)           | `#E2EFDA`  |

## Margens e posicionamento

Logomarcas, imagens, textos, crachá, cabeçalhos e rodapés ficam a
**0,5cm** das margens laterais (esquerda/direita), exceto a mini
logomarca da instituição no rodapé dos slides de questão, que fica
**centralizada** dentro dessa faixa de 0,5cm (não colada na borda).

As posições verticais e as dimensões das colunas de conteúdo do
slide de questão já foram conferidas e confirmadas — não são mais
um ajuste pendente. Cada coluna tem 15,75cm de largura (14,6cm de
altura nominal), com 1,0cm de espaço entre elas: canto superior
esquerdo da coluna da esquerda em (0,5cm; 3,4cm), da direita em
(17,25cm; 3,4cm). No uso automático (`\question`), o texto corrido
é cortado a 14,45cm — uma pequena folga em relação aos 14,6cm
nominais — para nunca encostar no rodapé.

## Faixas superior/inferior

Todo slide (título e questão) tem uma faixa verde-clara no topo e
outra, da mesma cor, na base (18,35cm a 19,05cm) — no slide de
questão a faixa inferior carrega o rodapé; no de título ela é só
decorativa, replicando o slide master do PPTX original.

## Fluxo de trabalho: template no GitHub, cada lista num clone separado

Este repositório fica **só com o template** — nunca uma lista de
exercícios real. Cada lista nova é um clone local independente:

```bash
# 1. Clonar o template num diretório novo para a lista
git clone <URL-deste-repositório> lista-vetores
cd lista-vetores

# 2. Remover o vínculo com este repositório (a lista não deve dar
#    push de volta pra cá — ela vira um projeto independente)
rm -rf .git

# 3. Editar config.tex: as 4 imagens (logo e ícone institucionais --
#    normalmente já apontam pro seu diretório fixo, não precisa mexer;
#    imagem de capa e logo do curso -- específicas desta lista) e os
#    metadados (\IFCEmetadados).
#
# 4. Editar main.tex: só as questões. Renomear se quiser
#    (ex.: lista-vetores.tex).

# 5. Compilar normalmente
xelatex main.tex && xelatex main.tex
```

Se quiser manter o histórico de versões da própria lista (não ligado
a este template), depois do passo 2 é só rodar `git init` de novo
nesse diretório e usar um repositório remoto próprio para ela — não
é obrigatório, mas funciona como um segundo repositório do zero.

Qualquer melhoria no template em si (novo recurso do `.sty`, ajuste
de posição, etc.) é feita **aqui**, neste repositório, nunca dentro
do clone de uma lista específica.

## Pendências conhecidas

Nenhuma pendência conhecida no momento. A continuação automática de
questão longa e o slide de gabarito (antes listados aqui) já foram
implementados — veja "Slide de questão" e "Gabarito" acima.
