# Guia de Execução: Terraform para AKS e Argo CD

Este guia descreve os passos para preparar, validar e aplicar as configurações de infraestrutura usando o Terraform no cluster AKS, incluindo o ajuste realizado para a integração do Argo CD.

---

## 1. Pré-requisitos
Antes de executar o Terraform, garanta que você está logado na sua conta da Azure e na assinatura correta:
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

Abra seu terminal e navegue até a pasta `cluster_aks`. Em seguida, execute a sequência abaixo:

### Passo A: Inicializar o Provedor e Módulos
Este comando baixa os novos provedores (`helm` e `kubernetes`) e o módulo local do `argocd`:
```powershell
terraform init
```

### Passo B: Validar o Código
Valida se as declarações, chaves e variáveis do código estão sintaticamente corretas:
```powershell
terraform validate
```

### Passo C: Planejar as Alterações
Gera uma prévia de todas as mudanças que serão aplicadas na Azure. 
```powershell
terraform plan
```
> [!NOTE]
> **Nota sobre o OIDC Issuer:**
> Durante o planejamento anterior, o Terraform tentou remover o OIDC Issuer (`oidc_issuer_enabled = true -> null`). Como a API do Azure não permite desativar esse recurso depois de ativado, adicionamos a linha `oidc_issuer_enabled = true` no arquivo `modules/aks/main.tf`. O plano agora deve rodar sem essa tentativa de remoção.

### Passo D: Aplicar e Criar os Recursos
Aplica as mudanças de infraestrutura e instala os novos recursos no Kubernetes:
```powershell
terraform apply
```
*Digite **`yes`** para confirmar.*

### Passo E: Adicionar o Contexto do AKS no Terminal
Antes de acessar o Argo CD ou gerenciar o cluster via `kubectl`, adicione o contexto do AKS ao seu terminal:
```powershell
az aks get-credentials --resource-group terraform-aks-rg --name terraform-aks-cluster
```

### Passo F: Destruir Recursos (Opcional)
Se precisar remover todos os recursos da Azure para evitar custos futuros:
```powershell
terraform destroy
```
*Digite **`yes`** para confirmar.*

---

## 3. Estrutura de Arquivos e Alterações

*   **[providers.tf](file:///c:/Users/pedro/Documentos/Teste/cluster_aks/providers.tf)**: Atualizado para suportar e configurar os provedores `kubernetes` e `helm` integrados às credenciais do AKS.
*   **[variables.tf](file:///c:/Users/pedro/Documentos/Teste/cluster_aks/variables.tf)**: Adicionadas variáveis globais de controle para o Argo CD (`enable_argocd`, `argocd_chart_version` e `argocd_helm_values`).
*   **[main.tf](file:///c:/Users/pedro/Documentos/Teste/cluster_aks/main.tf)**: Adicionada a chamada para instanciar o módulo `argocd`.
*   **[modules/aks/main.tf](file:///c:/Users/pedro/Documentos/Teste/cluster_aks/modules/aks/main.tf)**: Adicionada a flag `oidc_issuer_enabled = true` para compatibilidade com o recurso ativo na Azure.
*   **[modules/aks/outputs.tf](file:///c:/Users/pedro/Documentos/Teste/cluster_aks/modules/aks/outputs.tf)**: Exposição dos campos de certificado e host para autenticação do provedor de Kubernetes.
