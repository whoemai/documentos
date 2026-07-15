# Guia Passo a Passo: Login e Acesso à UI do Argo CD

Este guia explica como obter as credenciais de acesso e entrar no console de administração do Argo CD após a implantação do cluster.

---

## Passo 1: Conectar o Terminal ao AKS
Caso ainda não esteja conectado ao seu cluster AKS através do seu terminal local, autentique-se com o Azure CLI:
```powershell
az aks get-credentials --resource-group terraform-aks-rg --name terraform-aks-cluster
```

---

## Passo 2: Resgatar a Senha do Administrador
O Argo CD gera uma senha inicial criptografada e a salva no segredo `argocd-initial-admin-secret`. No terminal **PowerShell**, execute o comando abaixo para descriptografar e exibir a senha:

```powershell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}")))
```

Se você estiver usando o **Git Bash (MINGW64)**, Linux ou Mac, utilize este comando em vez do acima:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

*Copie a senha impressa no terminal.*

---

## Passo 3: Criar um Túnel Local (Port-Forward)
Como o serviço do Argo CD está protegido dentro da rede interna do cluster, você precisa expor a porta dele temporariamente na sua máquina:

```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
*Aviso: Este comando deve permanecer em execução para manter a interface no ar. Deixe esta janela do terminal aberta.*

---

## Passo 4: Acessar a Interface Gráfica
1. Abra seu navegador e navegue até: **[https://localhost:8080](https://localhost:8080)**.
2. Como o certificado SSL padrão do Argo CD é autoassinado, o navegador exibirá um alerta de segurança. Clique em **"Avançado"** e em seguida selecione a opção para prosseguir para o endereço.
3. Na tela de login, insira:
   * **Usuário (Username):** `admin`
   * **Senha (Password):** *[A senha descriptografada obtida no Passo 2]*
