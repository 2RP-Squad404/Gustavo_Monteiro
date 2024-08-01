# Git

**Aluno/Estagiário**: Gustavo Monteiro Gomes Pires
<br>
**Relatórios referentes a trilha de conhecimento:** "*Data_Science*"
<br>
**Realização:** 01/08/2024
<br>
**Data de Entrega:** 02/08/2024

## Introdução

Git é um sistema de versionamento de código, utilizado para rastrear alterações em arquivos e coordenar o trabalho de várias pessoas em projetos de desenvolvimento de software.

### Principais Conceitos do Git
- Repositório: É uma estrutura para organizar arquivos, também armazena o histórico completo de mudanças do projeto, incluindo arquivos(Ex: .png, .mp3, .md, etc.) e pastas.
- Commit: Como uma foto do projeto em um momento específico. Cada commit tem um identificador único (hash).
- Branch: Uma ramificação do projeto que permite trabalhar em diferentes linhas de desenvolvimento ao mesmo tempo.
- Merge: Combinação de mudanças de diferentes branches em uma única branch.
- Pull: Atualiza a cópia local do repositório com as mudanças mais recentes do repositório remoto.
- Push: Envia as mudanças locais para o repositório remoto.
- Clone: Criação de uma cópia local de um repositório remoto.
- Fork: Criação de uma cópia independente de um repositório, normalmente para contribuir com um projeto sem afetar o repositório original.

## Principais Componentes do GitFlow

### Branches Principais:
- **main**: Contém o código de produção,ou seja, a versão final onde o cliente tem acesso. Cada commit nesta branch representa uma nova versão de lançamento.
- **develop**: Contém o código em desenvolvimento com as últimas alterações. Serve como a base para novas funcionalidades e integrações.

### Branches de Suporte:
- **feature**: Usada para desenvolver novas funcionalidades do produto/código. Deriva de `develop` e é mesclada de volta a `develop` quando a funcionalidade está concluída.
- **release**: Serve para preparar o lançamento de uma nova versão. Deriva de `develop` e, após ajustes finais, é mesclada em `main` e `develop`.
- **hotfix**: Criada para correções urgentes em produção, por exemplo, problemas de segurança ou *bugs*. Deriva de `main` e é mesclada de volta em `main` e `develop` após a correção.

## Fluxo de Trabalho

### Desenvolvimento de Funcionalidades:
1. Crie uma branch de `feature` a partir de `develop`.
2. Desenvolva a nova funcionalidade.
3. Mescle a branch de `feature` de volta em `develop`.

### Preparação para Lançamento:
1. Crie uma branch de `release` a partir de `develop`.
2. Faça os ajustes finais e correções.
3. Mescle a branch de `release` em `main` e `develop`, e marque o commit com um tag.

### Correções Urgentes:
1. Crie uma branch de `hotfix` a partir de `main`.
2. Faça a correção necessária.
3. Mescle a branch de `hotfix` em `main` e `develop`, e marque o commit com um tag.