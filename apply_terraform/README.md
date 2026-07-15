# Guia de Execução: Terragrunt para AKS e Argo CD

Este guia descreve os passos para preparar, validar e aplicar as configurações de infraestrutura usando o Terragrunt no cluster AKS, integrando o deploy de aplicações (como o Argo CD) com gestão inteligente de dependências.

---

## 1. Pré-requisitos
Antes de executar o Terragrunt, garanta que você está logado na sua conta da Azure e na assinatura correta:
```powershell
az login
az account show --query "{name:name, id:id}"
```

Se precisar alterar a assinatura ativa:
```powershell
az account set --subscription <NOME_OU_ID_DA_ASSINATURA>
```

---

## 2. Passo a Passo dos Comandos

Abra seu terminal e navegue até a pasta `cluster_aks/environments`. Em seguida, execute a sequência abaixo:

### Passo A: Planejar as Alterações
O comando `run-all` varre todas as sub-pastas (ex: `infra` e `apps`) e monta o plano de execução inteiro respeitando a ordem de dependência (primeiro Infra, depois Apps).
```powershell
terragrunt run-all plan
```
> [!NOTE]
> **Nota sobre Dependências:**
> O módulo de `apps` tem uma dependência (`dependency "infra"`) e só será executado quando o cluster AKS da infra estiver provisionado e pronto para uso.

### Passo B: Aplicar e Criar os Recursos
Aplica as mudanças de infraestrutura e instala os recursos no Kubernetes de uma só vez:
```powershell
terragrunt run-all apply
```
*Digite **`y`** para confirmar.*

### Passo C: Adicionar o Contexto do AKS no Terminal
Antes de acessar o Argo CD ou gerenciar o cluster via `kubectl`, adicione o contexto do AKS ao seu terminal:
```powershell
az aks get-credentials --resource-group rg-aks-dev --name terragrunt-aks-dev
```

### Passo D: Destruir Recursos (Opcional)
A maior vantagem da separação de estados do Terragrunt. Para excluir **tudo** de forma segura (ele apagará os Apps primeiro, depois a Infraestrutura):
```powershell
terragrunt run-all destroy
```
*Digite **`y`** para confirmar.*

---

## 3. Estrutura de Arquivos e Alterações

A nova estrutura adota o padrão DRY (Don't Repeat Yourself) e separa o código dos valores de ambiente:

*   **`modules/`**: Módulos Terraform "puros" (`infra` e `apps`).
*   **`environments/terragrunt.hcl`**: Arquivo mestre que injeta dinamicamente os providers.
*   **`environments/dev/env.hcl`**: Contém as variáveis globais do ambiente.
*   **`environments/dev/infra/terragrunt.hcl`**: Realiza o deploy da infraestrutura.
*   **`environments/dev/apps/terragrunt.hcl`**: Faz o deploy do ArgoCD possuindo dependência declarada com o módulo de infraestrutura.
