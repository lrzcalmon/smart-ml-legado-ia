# SMART – estudo de caso de um sistema legado

> Documentação pública de um sistema legado real, usada como exemplo em um artigo sobre **como lidar com legados**.
> O conteúdo é conceitual: descreve o que o sistema faz, como foi construído e o que a análise revelou, sem expor
> código, algoritmos, sintaxes internas nem mecanismos de proteção.
>
> | Documento | Conteúdo |
> |---|---|
> | [ARCHITECTURE.md](ARCHITECTURE.md) | Arquitetura em alto nível, modelo de dados conceitual, riscos e caminhos de modernização |
> | [Manual.pdf](Manual.pdf) | Manual do usuário com as telas reconstruídas |
> | [telas/](telas/) | Telas reconstruídas a partir dos formulários originais |

![Tela principal reconstruída](telas/FMaster.svg)

## 1. O sistema

O **SMART** – *Source code Maintenance, Analysis and Reestructuring Tools* – é uma ferramenta desktop Windows criada
pela **Millennium Consultoria e Informática Ltda** para **corrigir, atualizar e dar manutenção em código-fonte em larga
escala**.

- **Origem: o B2k (bug do ano 2000).** O objetivo inicial era localizar, em milhares de programas, os pontos que
  tratavam o ano com dois dígitos e aplicar as correções de forma controlada, auditada e repetível.
- **Independente de linguagem.** Cada linguagem é descrita por uma configuração própria. A base de exemplo trazia
  cerca de vinte linguagens configuradas: variantes de COBOL, CICS, JCL, geradores de aplicação de mainframe,
  linguagens xBase, Visual Basic, C/C++ e linguagens web.
- **Pseudo-linguagem própria.** A ferramenta tem uma linguagem interna para descrever linguagens, expressar
  pesquisas e representar as operações de manutenção (ver [ARCHITECTURE.md](ARCHITECTURE.md#5-a-pseudo-linguagem)).
- **Inteligência própria.** Entende a estrutura de cada linha (áreas de controle, código, comentários, continuação),
  pesquisa com operadores lógicos, respeita limites da linguagem e aplica regras de negócio antes de cada alteração.
- **Componentes próprios.** Foi desenvolvido um conjunto de componentes Delphi específico para a ferramenta.
- **Proteção.** A licença era vinculada à máquina por criptografia baseada em dados internos do equipamento, com
  mecanismos complementares de verificação de cópia oficial.

### Funcionalidades

| Área | O que faz |
|---|---|
| Administração | Parametrização de linguagens, cadastro de sistemas e carga de programas, usuários e permissões, vínculo usuário × sistema |
| Manutenção | Pesquisa em lote, pesquisa com atualização assistida, edição livre de fontes |
| Controle | Estatísticas de produtividade por sistema, usuário e período |
| Exportação | Promoção controlada por etapas: compilação, testes, homologação e produção |
| RollBack | Consulta e restauração de versões anteriores |

Funcionalidades previstas mas não concluídas no código encontrado: importação de arquivos, controle de mudanças,
referência cruzada e dicionário de dados.

## 2. Linha do tempo (inferida pelos artefatos)

| Período | Evidência |
|---|---|
| 1998–1999 | Motor voltado ao B2k; primeira geração sobre banco de dados de arquivos locais |
| 2000 | Documento de liberação do produto para comercialização |
| 2003–2005 | Ajuda on-line, listas de pendências para novas versões do Windows, migração do acesso a dados |
| 2011 | Última recompilação do executável |
| Depois | Tentativa de conversão automática para outro compilador, abortada no meio |

## 3. Tecnologia

| Item | Tecnologia |
|---|---|
| Linguagem | Object Pascal (Delphi, VCL) |
| Banco de dados | Microsoft Access via ADO |
| Armazenamento de fontes | Arquivos texto em área protegida, com cópias de trabalho e versões |
| Extensibilidade | Plugins em DLL para os diálogos de parametrização de linguagem |
| Ajuda | WinHelp |

## 4. O que a análise encontrou

Estado do material recebido para análise:

- Código-fonte, binários compilados, banco de dados, ajuda, imagens e documentos de negócio misturados na mesma
  árvore, **sem controle de versão**.
- Partes importantes **só em forma compilada** (componentes e units da geração anterior).
- **Conversão de compilador interrompida**, deixando arquivos em dois encodings e o projeto original fora do lugar.
- Caminhos absolutos e configurações gravados no código.
- Funcionalidades desligadas por linhas comentadas, incluindo parte da proteção.
- Documentação de usuário em formato que os sistemas operacionais atuais não abrem mais.

## 5. Como a documentação foi reconstruída

1. **Inventário** de todas as pastas e tipos de arquivo, separando produto, protótipos, ferramentas de terceiros e
   artefatos de build.
2. **Leitura dirigida do código**: ponto de entrada, módulo de dados, unidades globais e um exemplo de cada padrão
   (plugin, formulário, regra).
3. **Reconstrução das telas** a partir dos arquivos de formulário (`.dfm`/`.lfm`), em texto e em binário, sem executar
   o sistema. Isso evitou efeitos colaterais na máquina de análise e reproduziu posição, textos, menus e ícones
   originais.
4. **Extração de regras de negócio** das validações e mensagens exibidas ao usuário.
5. **Leitura do banco sem o driver original**, extraindo do arquivo os catálogos de parâmetros e as linguagens
   configuradas.
6. **Validação com quem conhecia o sistema**: as hipóteses do código foram confrontadas com o relato do autor, e cada
   afirmação foi marcada como confirmada, parcial ou não encontrada.

## 6. Lições para o artigo

- **O legado tem mais conhecimento do que código.** As regras de negócio estavam espalhadas em mensagens, validações e
  documentos de projeto, não em uma especificação.
- **Formulários são documentação.** Em sistemas RAD, os arquivos de tela guardam a interface completa e permitem
  produzir um manual sem rodar o programa.
- **Desconfie dos binários soltos.** Componentes só compilados são dependências que ninguém consegue mais reconstruir.
- **Migrações automáticas pela metade são piores que nenhuma.** O primeiro passo é restaurar um build reproduzível.
- **Valide com as pessoas.** Parte das afirmações sobre o sistema só se confirmou parcialmente no código, e outras
  dependiam de contexto que só o autor tinha.
- **Separe o público do interno.** A versão pública descreve conceitos; algoritmos, sintaxes e proteção ficam na
  documentação interna.
