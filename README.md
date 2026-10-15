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
├── main.tex                       # exemplo/ponto de partida (capa + 2 questões, tudo genérico)
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

### Ativos institucionais — `\IFCEativosInstitucionais{logo}{ícone}` (uma vez, no preâmbulo)

```latex
\IFCEativosInstitucionais
  {\includegraphics[width=7.25cm]{/caminho/externo/logo-ifce-baturite.pdf}}
  {\includegraphics[width=0.37cm]{/caminho/externo/icone-ifce-mini.pdf}}
```

A logomarca institucional (capa) e o ícone minúsculo (rodapé dos
slides de questão) são **sempre os mesmos**, em qualquer lista — mas
os arquivos de imagem não vêm com o template, ficam num diretório
externo, fora do controle de versão (veja "Fluxo de trabalho"). Esses
dois campos são conteúdo livre, como o `\IFCEplaceholder` usado no
`main.tex` de demonstração.

### Metadados globais — `\IFCEmetadados` (uma vez, no preâmbulo)

```latex
\IFCEmetadados
  {IFCE Baturité}              % instituição
  {<curso/turma>}               % curso/turma
  {<nome da disciplina>}        % disciplina
  {<tipo de conteúdo>}          % tipo de conteúdo
  {<descrição do conteúdo>}     % descrição do conteúdo
  {<assunto/tópico>}            % assunto/tópico
  {<nome do professor>}         % professor
  {\includegraphics{...}}       % logomarca do curso (8º campo)
```

Esses oito campos alimentam **automaticamente** o slide de título e o
cabeçalho/rodapé de todo slide de questão — não são repetidos em cada
comando.

O **8º campo** (logomarca do curso) é conteúdo livre e fica de fora do
template-base de propósito: cada curso tem a sua própria arte, então
esse não é um elemento institucional fixo. Use
`\IFCEplaceholder[2.0cm]{logo do curso}` (como no `main.tex` de
demonstração) até ter a arte definitiva; numa lista de exercícios real,
troque por `\includegraphics{caminho/da/logo-do-curso}`. O nome do
curso (2º campo) continua sendo só texto.

Os campos `<disciplina>`, `<tipo de conteúdo>` e `<assunto>` aparecem
separados pelo símbolo "▪" (`\IFCEsep`, desenhado com `\rule` — não
depende do glifo existir na fonte) tanto na faixa superior do slide de
questão quanto, com instituição/curso/professor, no rodapé.

### Slide de título — `\CapaIFCE{conteúdo à direita}`

```latex
\CapaIFCE{\IFCEplaceholder{imagem de capa}}
```

Só recebe a ilustração da capa; todo o resto vem de
`\IFCEativosInstitucionais` e `\IFCEmetadados`.

### Slide de questão — `\question{título/edição}{coluna esquerda}{coluna direita}`

```latex
\question
  {<edição/informação da questão>}   % ex.: "ENEM 2023", "UECE 2022.1"
  {<enunciado da questão>}
  {<conteúdo/imagem/resposta>}
```

O número do crachá avança sozinho a cada `\question` — `"01"`, `"02"`,
... — e sua linha de base é **sempre igual** à do título ao lado (os
dois nós do TikZ são ancorados na mesma coordenada de baseline, então
o alinhamento é garantido por construção, não por ajuste visual). O
cabeçalho da faixa superior (`<disciplina> ▪ <tipo de conteúdo> ▪
<assunto>`) e o rodapé (`<instituição> ▪ <curso> ▪ Prof. <professor>`
à esquerda, `Slide X de N` à direita) são montados automaticamente a
partir dos metadados globais.

As duas colunas de conteúdo têm dimensões **exatas e fixas**: 16,0 ×
14,6cm cada, com cantos superiores esquerdos em (0,5cm; 3,4cm) e
(17,0cm; 3,4cm), respectivamente. A **primeira coluna** (esquerda)
recebe um recuo de **2,0cm na primeira linha de cada parágrafo**
(`\parindent` local, com `\indent` explícito logo no início — dentro
de um nó do TikZ a primeira linha não herda o recuo automático de
parágrafo, só as continuações depois de linha em branco, por isso o
`\indent` extra é necessário); a segunda coluna não tem recuo.

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
slide de questão (16,0 × 14,6cm cada, ver acima) já foram conferidas
e confirmadas — não são mais um ajuste pendente.

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

# 3. Editar main.tex:
#    - \IFCEativosInstitucionais: caminho para a logo e o ícone
#      institucionais (ficam guardados num diretório fixo seu, fora
#      de qualquer repositório — a mesma logomarca serve pra toda
#      lista, então não precisa copiá-la a cada clone);
#    - \IFCEmetadados: metadados da lista + logomarca do curso
#      (8º campo, específica desse curso);
#    - as questões.
#    Renomear main.tex se quiser (ex.: lista-vetores.tex).

# 4. Compilar normalmente
xelatex main.tex && xelatex main.tex
```

Se quiser manter o histórico de versões da própria lista (não ligado
a este template), depois do passo 2 é só rodar `git init` de novo
nesse diretório e usar um repositório remoto próprio para ela — não
é obrigatório, mas funciona como um segundo repositório do zero.

Qualquer melhoria no template em si (novo recurso do `.sty`, ajuste
de posição, etc.) é feita **aqui**, neste repositório, nunca dentro
do clone de uma lista específica.

## Pendências conhecidas (propositalmente fora deste template ainda)

- **Continuação automática de questão longa** (`[continua>]` /
  `[continuação]`, inserida sozinha quando o conteúdo não cabe em um
  slide): fica para uma próxima rodada. A ideia é usar `\vsplit` do
  TeX para medir onde o conteúdo "estoura" a altura fixa da coluna e
  cortar ali — funciona bem para texto corrido, com a ressalva de que
  uma imagem grande bem no ponto de corte pode exigir ajuste manual
  pontual.
- **Slide de gabarito**: a classe `exam` (com `\choice`,
  `\CorrectChoice`, etc.) é incompatível com `\documentclass{beamer}`
  — são duas classes diferentes, não dá pra combinar. A alternativa
  equivalente é um mecanismo próprio: cada `\question` grava a
  resposta certa num arquivo auxiliar (como bibliografia/sumário
  fazem), e um comando `\gabaritoIFCE` no fim do documento lê esse
  arquivo e monta a tabela sozinho. Ainda não implementado no
  template — uma versão manual (`\GabaritoIFCE`, sem contador
  automático) já foi usada numa lista específica como solução
  provisória, fora deste repositório.
