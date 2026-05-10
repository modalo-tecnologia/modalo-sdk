# modalo-sdk
Configuração do repositório.

# Como configurar

1. Criar o "Receptor" Git (Bare Repository) no Servidor
O Git não joga os arquivos direto na pasta final, ele recebe em um cofre especial e depois nós movemos. Rode isso no VPS:
```bash
# Crie uma pasta para guardar o "cofre" do git (exemplo em /var/git)
mkdir -p .var/git/modalo.git
cd .var/git/modalo.git
# Inicie um repositório "recebedor"
git init --bare
```

2. Criar o "Gatilho" Automático (O Hook)
Ainda no VPS, dentro da pasta .var/git/modalo.git/hooks/, crie um arquivo chamado post-receive.

É esse cara que substitui o deploy.js. Ele é ativado sozinho quando você manda um código novo pra ele. Escreva dentro desse arquivo (use nano hooks/post-receive):

```bash
#!/bin/bash
echo "📦 Recebendo novo código na Modalo..."
# Para onde o código real vai ser descompactado (sua pasta de produção)
TARGET_DIR=".var/www/modalo-producao"
# Descompacta o código que acabou de chegar
git --work-tree=$TARGET_DIR --git-dir=/var/git/modalo.git checkout -f
# Vai até a pasta de produção e roda as coisas
cd $TARGET_DIR
echo "⚙️ Instalando pacotes com Bun..."
bun install
# Reinicia o Dashboard/Backend ou o serviço desejado
echo "🔄 Reiniciando serviços..."
pm2 restart modalo-api || pm2 start backend.js --name "modalo-api"
echo "✅ Deploy concluído via Git!"
```

3. Dar Permissão Pro Gatilho
Rode isso no servidor para ele poder ser executado:

```bash
chmod +x hooks/post-receive
```
