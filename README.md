# -MEGA-DOWNLOAD-BR-

Um projeto Brasileiro via Termux para baixar vídeos e áudios de qualquer site ou plataforma.

---

## 🚀 Como Instalar e Usar no Termux

Siga os dois passos abaixo dentro do seu Termux. Copie um comando por vez e cole no aplicativo.

### 🛠️ Passo 1: Dar permissão de armazenamento
Este comando garante que o Termux consiga salvar os vídeos na pasta de Downloads do seu celular. Copie, cole no Termux e dê Enter (se aparecer uma mensagem na tela pedindo permissão, clique em "Permitir"):

```bash
termux-setup-storage
```
⚡ Passo 2: Instalação e Configuração Automatizada
Agora, copie o bloco de código gigante abaixo por completo, cole no seu Termux e dê Enter. Ele vai atualizar o sistema, instalar o yt-dlp, o ffmpeg e criar o script de inicialização automaticamente se aparecer mais uma tela pedindo para você digitar y ou n basta digitar " y " para sim:
```bash
export DEBIAN_FRONTEND=noninteractive && pkg update -y -o Dpkg::Options::="--force-confnew" && pkg upgrade -y -o Dpkg::Options::="--force-confnew" && pkg install ffmpeg python python-pip termux-api ca-certificates ncurses-utils -y && pip install curl-cffi yt-dlp && termux-setup-storage && cat << 'EOF' > baixar
#!/bin/bash

# SISTEMA DE CORES HACKER
VERDE="\e[1;32m"
VERDE_CLARO="\e[1;92m"
CIANO="\e[1;36m"
AMARELO="\e[1;33m"
VERMELHO="\e[1;31m"
RESET="\e[0m"

# GARANTE A CRIAÇÃO DA PASTA CORRETA (D Maiúsculo)
PASTA_DESTINO="/storage/emulated/0/Documents/Download"
mkdir -p "$PASTA_DESTINO"

# FUNÇÃO MATRIX REALISTA DE 10 SEGUNDOS (125 ciclos x 0.08s = 10 segundos)
matrix_effect() {
    clear
    echo -e "${VERDE}"
    for i in {1..125}; do
        echo "01101001 01110011 01101111 00100000 01100101 01101101 01010100 01000101"
        sleep 0.08
    done
    echo -e "${RESET}"
}

# LOOP DO MENU PRINCIPAL
while true; do
    clear
    echo -e "${CIANO}==================================================${RESET}"
    echo -e "${VERDE}[BR] ${VERDE_CLARO}🇧🇷  MEGA DOWNLOAD BR  🇧🇷 ${AMARELO}[BR]${RESET}"
    echo -e "${CIANO}==================================================${RESET}"
    echo ""
    echo -e "${AMARELO}👉 DIGITE A PALAVRA-CHAVE [BAIXAR] PARA INICIAR${RESET}"
    echo -e "${VERMELHO}👉 DIGITE [SAIR] PARA FECHAR O PROGRAMA${RESET}"
    echo ""
    read -p "SISTEMA > " COMANDO

    COMANDO_LOWER=$(echo "$COMANDO" | tr '[:upper:]' '[:lower:]')

    if [ "$COMANDO_LOWER" = "sair" ]; then
        echo -e "\n${VERDE}Saindo do MEGA DOWNLOAD BR... Até logo!${RESET}\n"
        break
    fi

    if [ "$COMANDO_LOWER" = "baixar" ]; then
        clear
        echo -e "${CIANO}==================================================${RESET}"
        echo -e "${VERDE}             PAINEL DE CAPTURA HACKER             ${RESET}"
        echo -e "${CIANO}==================================================${RESET}"
        echo ""
        
        read -p "🔗 COLE A URL DO SEU ALVO AQUI: " URL

        URL=$(echo "$URL" | xargs)

        if [ -z "$URL" ]; then
            echo -e "\n${VERMELHO}❌ ERRO: NENHUMA URL DETECTADA!${RESET}"
            echo -e "${AMARELO}Escreva VOLTAR para o menu principal.${RESET}"
            read -p "> " RETORNO
            continue
        fi

        # SELEÇÃO TOTALMENTE EM AZUL CIANO DESTACADO
        echo ""
        echo -e "${CIANO}==================================================${RESET}"
        echo -e "${CIANO}               SELECIONE A FUNÇÃO                 ${RESET}"
        echo -e "${CIANO}==================================================${RESET}"
        echo -e "${CIANO} [ 1 ] BAIXAR VÍDEO (AVC1 + MP4A / MÁXIMA QUALIDADE)${RESET}"
        echo -e "${CIANO} [ 2 ] BAIXAR ÁUDIO (MP3 / EXTRAÇÃO DIRETA NATIVA)  ${RESET}"
        echo -e "${CIANO}==================================================${RESET}"
        read -p "ESCOLHA UMA OPÇÃO (1 OU 2): " OPCAO

        echo ""

        # VERIFICA SE A URL É DO SITE ALVO PROTEGIDO
        if [[ "$URL" =~ pornhub\.com ]]; then
            IS_PORNHUB=true
        else
            IS_PORNHUB=false
        fi

        case $OPCAO in
            1)
                echo -e "${VERDE}⚡ INICIANDO PROTOCOLO DE EXTRAÇÃO DE VÍDEO...${RESET}"
                
                if [ "$IS_PORNHUB" = true ]; then
                    # Usa o disfarce (--impersonate chrome) que fez funcionar no seu teste
                    yt-dlp --impersonate chrome -f "best" -P "$PASTA_DESTINO" "$URL"
                else
                    # Mantém o seu comando padrão intocado para TikTok, Instagram, etc.
                    yt-dlp -f "bestvideo[vcodec^=avc1]+bestaudio[acodec^=mp4a]/best[vcodec^=avc1]/best" --merge-output-format mp4 -P "$PASTA_DESTINO" "$URL"
                fi

                if [ $? -eq 0 ]; then
                    matrix_effect
                    echo -e "${VERDE}✔ VÍDEO BAIXADO E COMPILADO COM SUCESSO!${RESET}"
                    echo -e "${AMARELO}Salvo em: Documents/Download${RESET}"
                else
                    echo -e "\n${VERMELHO}❌ OCORREU UM ERRO NO PROCESSAMENTO DO VÍDEO ACIMA!${RESET}"
                    echo -e "${AMARELO}Verifique o link ou a sua conexão de rede.${RESET}"
                fi
                ;;
            2)
                echo -e "${VERDE}⚡ INICIANDO PROTOCOLO DE EXTRAÇÃO DE ÁUDIO...${RESET}"
                
                if [ "$IS_PORNHUB" = true ]; then
                    # Aplica o disfarce também no modo áudio para não tomar bloqueio
                    yt-dlp --impersonate chrome -x --audio-format mp3 -P "$PASTA_DESTINO" "$URL"
                else
                    yt-dlp -x --audio-format mp3 -P "$PASTA_DESTINO" "$URL"
                fi

                if [ $? -eq 0 ]; then
                    matrix_effect
                    echo -e "${VERDE}✔ ÁUDIO CONVERTIDO EM MP3 COM SUCESSO!${RESET}"
                    echo -e "${AMARELO}Salvo em: Documents/Download${RESET}"
                else
                    echo -e "\n${VERMELHO}❌ OCORREU UM ERRO NA EXTRAÇÃO DO ÁUDIO ACIMA!${RESET}"
                    echo -e "${AMARELO}Verifique o link ou a sua conexão de rede.${RESET}"
                fi
                ;;
            *)
                echo -e "${VERMELHO}❌ OPÇÃO INVÁLIDA NO SISTEMA!${RESET}"
                ;;
        esac

        # RETORNO AO MENU
        echo ""
        echo -e "${CIANO}==================================================${RESET}"
        echo -e "${AMARELO}DIGITE [VOLTAR] PARA RETORNAR AO MENU INICIAL${RESET}"
        echo -e "${CIANO}==================================================${RESET}"
        
        while true; do
            read -p "> " VOLTA_MENU
            VOLTA_LOWER=$(echo "$VOLTA_MENU" | tr '[:upper:]' '[:lower:]')
            if [ "$VOLTA_LOWER" = "voltar" ]; then
                break
            fi
            echo -e "${VERMELHO}Comando inválido. Digite exatamente VOLTAR:${RESET}"
        done

    else
        echo -e "\n${VERMELHO}❌ PALAVRA-CHAVE INVÁLIDA! USE 'BAIXAR' OU 'SAIR'.${RESET}"
        sleep 2
    fi
done
EOF
chmod +x baixar && mv baixar $PREFIX/bin/ && clear && echo -e "${VERDE}=== 🇧🇷 MEGA DOWNLOAD BR INSTALADO CORRETAMENTE! 🇧🇷 ===${RESET}\n\nDigite apenas:\n\nbaixar\n"

```
📱 Como abrir o programa depois de instalado?
Sempre que abrir o Termux, basta digitar:
```bash
baixar
