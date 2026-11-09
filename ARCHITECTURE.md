# Arquitetura – SMART (visão pública)

> Visão de alto nível de um sistema legado usado como estudo de caso. Detalhes de implementação, sintaxes internas,
> algoritmos e mecanismos de proteção foram omitidos de propósito.
> Ver também: [README.md](README.md) · [Manual.pdf](Manual.pdf)

## 1. Estilo arquitetural

Aplicação **desktop monolítica (fat client)** em Delphi/VCL, no padrão RAD *formulários + módulo de dados*:

- regras de negócio, acesso a dados e interface convivem nos mesmos formulários;
- um único módulo de dados concentra todas as tabelas, compartilhado por toda a aplicação;
- estado global extenso em unidades utilitárias;
- **persistência híbrida**: metadados e histórico em banco Access, conteúdo dos fontes em arquivos texto;
- **plugins em DLL** para os diálogos de parametrização das linguagens.

```
┌──────────────────────────────────────────────────────────────┐
│                     Aplicação desktop (VCL)                  │
│                                                              │
│  Login ─▶ Janela principal (menu, barra, navegador, F-keys)  │
│             ├─ Administração ──────▶ Plugins de linguagem    │
│             ├─ Manutenção (pesquisa / atualização / edição)  │
│             ├─ Controle (estatísticas)                       │
│             ├─ Exportação (ciclo de vida)                    │
│             └─ RollBack                                      │
│                                                              │
│   Núcleo: modelo de linha · motor de pesquisa · motor de     │
│           atualização · log/auditoria · versionamento        │
└───────────────┬───────────────────────────────┬──────────────┘
                │                               │
        ┌───────▼────────┐            ┌─────────▼─────────────┐
        │ Banco Access   │            │ Área protegida de     │
        │ metadados, log,│            │ fontes: original,     │
        │ usuários,      │            │ referência, trabalho, │
        │ versões        │            │ versões exportadas    │
        └────────────────┘            └───────────────────────┘
```

## 2. Componentes

| Componente | Papel |
|---|---|
| Aplicação principal | Interface, regras e orquestração |
| Plugins de linguagem (≈20 DLLs) | Um diálogo por tipo de parâmetro de linguagem, todos derivados de um formulário-base comum |
| Componentes Delphi próprios | Pacote de componentes visuais e utilitários criados para a ferramenta |
| Utilitários de suporte | Gerador de licença, recuperação de estado de execução |
| Ajuda on-line | Arquivo de ajuda vinculado às telas |

## 3. Modelo de dados (conceitual)

```
Linguagem 1─N Programa N─1 Sistema N─N Usuário
                 │              │
                 1              1
                 │              │
                 N              N
         Linhas alteradas     Versões
                 │
                 └──── Log de operações (usuário, sessão, serviço, data)

Catálogo de parâmetros ─▶ define quais características de uma linguagem são configuráveis
```

| Entidade | Papel |
|---|---|
| Sistema | Aplicativo do cliente, com versão, release e status no ciclo de vida |
| Programa | Fonte de um sistema, com linguagem, métricas (linhas de código, comentário, branco) e status |
| Linguagem | Conjunto de parâmetros que descreve como ler uma linha daquela linguagem |
| Catálogo de parâmetros | Lista dos parâmetros configuráveis e do diálogo responsável por cada um |
| Linhas | Histórico das linhas originais e alteradas por versão |
| Versões | Versões exportadas e programas contidos |
| Usuário / Vínculo | Funções habilitadas e sistemas visíveis, com nível normal ou privilegiado |
| Sessão / Log | Auditoria completa das operações |

Integridade referencial e exclusões em cascata eram feitas pela aplicação, não pelo banco.

## 4. Armazenamento de fontes

Cada programa carregado mantém cópias separadas:

| Cópia | Finalidade |
|---|---|
| Original | Nunca é alterada |
| Referência | Base de comparação para gerar diferenças |
| Trabalho | Recebe as alterações |
| Versões | Uma pasta por versão exportada, usada pelo RollBack |

Áreas de controle das linhas (por exemplo, numeração e identificação de colunas fixas) são guardadas à parte do corpo
do código, para que nunca sejam alteradas por engano.

## 5. A pseudo-linguagem

O núcleo da ferramenta é uma linguagem interna em três níveis:

1. **Descrição de linguagens.** Expressões compactas dizem onde fica o código útil em cada linha, como se
   reconhecem comentários, continuações e inclusões de arquivos, quais extensões pertencem à linguagem e se ela
   diferencia maiúsculas. Com isso, a mesma ferramenta atende de COBOL de colunas fixas a linguagens web de formato
   livre.
2. **Pesquisa.** Termos combinados com operadores lógicos, critérios de palavra inteira e tratamento de espaços,
   avaliados apenas sobre a área de código (comentários e áreas de controle são respeitados).
3. **Execução.** Cada atualização é representada por um modelo lógico que combina **alcance** (todos os programas,
   um programa, uma linha), **referência** (o termo ou a linha) e **operação** (substituir, incluir, excluir). Sobre
   esse modelo foi definido um conjunto de instruções de mais alto nível, incluindo as transformações de datas do B2k.

Uma evolução planejada previa escrever rotinas uma única vez, em algoritmo próprio, e gerá-las automaticamente em
cada linguagem-alvo.

## 6. Fluxos principais

| Fluxo | Resumo |
|---|---|
| Carga | Cadastro do sistema → seleção de arquivos → validação (texto, tamanho de linha, duplicidade) → cópias na área protegida → métricas |
| Pesquisa | Sistema e programas → termos e operadores → varredura respeitando a estrutura da linha → navegação pelas linhas impactadas |
| Atualização | Linha impactada → escolha de alcance, referência e operação → execução seletiva ou completa → nova versão, log e relatório de exceções |
| Exportação | Alterado → Compilação → Testes → Homologação → Produção (somente administrador), com versão ou release incrementados |
| RollBack | Cópia de consulta de versões anteriores ou restauração da versão corrente, descartando alterações não exportadas |

## 7. Segurança e proteção

- Funções habilitadas individualmente por usuário; menus e botões refletem as permissões.
- Segregação de funções: o perfil administrativo de fábrica administra e promove para produção, mas não altera código.
- Usuários vinculados a sistemas, com nível normal ou privilegiado; programas podem ser marcados como restritos.
- Sessão numerada, amarrada à estação, com log de entrada e saída.
- Licença vinculada à máquina por criptografia baseada em dados internos do equipamento, com ativação por
  desafio e resposta junto ao fornecedor.
- Mecanismos complementares de captura de exceções e de eventos da aplicação, usados também para garantir a execução
  de cópia oficial.

## 8. Riscos e débitos técnicos

| Área | Situação encontrada |
|---|---|
| Segurança | Técnicas de proteção de senha e de licença adequadas à época, mas abaixo do padrão atual; consultas montadas por concatenação |
| Plataforma | Dependência de drivers e formatos de ajuda descontinuados, execução restrita a 32 bits |
| Dados | Banco de arquivo único, integridade apenas na aplicação, conteúdo duplicado entre arquivos e tabelas |
| Código | Estado global, formulários extensos misturando UI e regra, duplicação entre módulos, dependências circulares |
| Build | Sem controle de versão, binários junto do código, encodings misturados, caminhos absolutos |
| Conhecimento | Componentes só compilados, regras não documentadas fora do código, funcionalidades desligadas sem registro |

## 9. Caminhos de modernização

1. Restaurar um build reproduzível e colocar tudo sob controle de versão.
2. Extrair o núcleo de domínio (modelo de linha, pesquisa, atualização) para módulos testáveis, independentes da UI.
3. Cobrir o núcleo com testes de caracterização usando os fontes de exemplo, antes de qualquer reescrita.
4. Trocar o banco por um SGBD com integridade referencial e parametrizar o acesso a dados.
5. Substituir os plugins binários por configuração declarativa versionável.
6. Revisar autenticação, permissões e licenciamento conforme práticas atuais.
