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
export DEBIAN_FRONTEND=noninteractive && pkg update -y -o Dpkg::Options::="--force-confnew" && pkg upgrade -y -o Dpkg::Options::="--force-confnew" && pkg install ffmpeg curl ncurses-utils ca-certificates python python-pip -y && pip install --upgrade "yt-dlp[curl-cffi]" && cat << 'EOF' > baixar
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

# FUNÇÃO MATRIX REALISTA DE 10 SEGUNDOS
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
    echo -e "${AMARELO}👉 DIGITE [ATUALIZAR] PARA ATUALIZAR O YT-DLP + IMPERSONAÇÃO${RESET}"
    echo -e "${VERMELHO}👉 DIGITE [SAIR] PARA FECHAR O PROGRAMA${RESET}"
    echo ""
    read -p "SISTEMA > " COMANDO

    COMANDO_LOWER=$(echo "$COMANDO" | tr '[:upper:]' '[:lower:]')

    if [ "$COMANDO_LOWER" = "sair" ]; then
        echo -e "\n${VERDE}Saindo do MEGA DOWNLOAD BR... Até logo!${RESET}\n"
        break
    fi

    if [ "$COMANDO_LOWER" = "atualizar" ]; then
        echo -e "\n${VERDE}⚡ ATUALIZANDO O YT-DLP E MÓDULOS DE IMPERSONAÇÃO...${RESET}"
        pip install --upgrade "yt-dlp[curl-cffi]"
        echo -e "${VERDE}✔ ATUALIZAÇÃO CONCLUÍDA COM SUCESSO!${RESET}"
        sleep 2
        continue
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

        echo ""
        echo -e "${CIANO}==================================================${RESET}"
        echo -e "${CIANO}               SELECIONE LA FUNÇÃO                 ${RESET}"
        echo -e "${CIANO}==================================================${RESET}"
        echo -e "${CIANO} [ 1 ] BAIXAR VÍDEO (MÁXIMA QUALIDADE / MP4 GALERIA)   ${RESET}"
        echo -e "${CIANO} [ 2 ] BAIXAR ÁUDIO (MP3 / EXTRAÇÃO DIRETA NATIVA)     ${RESET}"
        echo -e "${CIANO}==================================================${RESET}"
        read -p "ESCOLHA UMA OPÇÃO (1 OU 2): " OPCAO

        echo ""

        case $OPCAO in
            1)
                echo -e "${VERDE}⚡ INICIANDO PROTOCOLO DE EXTRAÇÃO DE VÍDEO...${RESET}"
                # Baixa na melhor qualidade priorizando codecs universais H.264/AAC nativos do MP4
                if yt-dlp -f "bv*[vcodec^=avc]+ba[acodec^=mp4a]/bv*+ba/best" -S "vcodec:h264,res,acodec:m4a" --merge-output-format mp4 --no-mtime -P "$PASTA_DESTINO" "$URL"; then
                    matrix_effect
                    echo -e "${VERDE}✔ VÍDEO BAIXADO COM SUCESSO!${RESET}"
                    echo -e "${AMARELO}Salvo em: Documents/Download${RESET}"
                else
                    echo -e "\n${VERMELHO}❌ OCORREU UM ERRO NO PROCESSAMENTO DO VÍDEO ACIMA!${RESET}"
                    echo -e "${AMARELO}Dica: Digite 'atualizar' no menu principal para atualizar as dependências.${RESET}"
                fi
                ;;
            2)
                echo -e "${VERDE}⚡ INICIANDO PROTOCOLO DE EXTRAÇÃO DE ÁUDIO...${RESET}"
                if yt-dlp -x --audio-format mp3 --no-mtime -P "$PASTA_DESTINO" "$URL"; then
                    matrix_effect
                    echo -e "${VERDE}✔ ÁUDIO CONVERTIDO EM MP3 COM SUCESSO!${RESET}"
                    echo -e "${AMARELO}Salvo em: Documents/Download${RESET}"
                else
                    echo -e "\n${VERMELHO}❌ OCORREU UM ERRO NA EXTRAÇÃO DO ÁUDIO ACIMA!${RESET}"
                    echo -e "${AMARELO}Dica: Digite 'atualizar' no menu principal para atualizar as dependências.${RESET}"
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
        echo -e "\n${VERMELHO}❌ PALAVRA-CHAVE INVÁLIDA! USE 'BAIXAR', 'ATUALIZAR' OU 'SAIR'.${RESET}"
        sleep 2
    fi
done
EOF
chmod +x baixar && mv baixar $PREFIX/bin/ && clear && echo -e "${VERDE}=== 🇧🇷 MEGA DOWNLOAD BR CORRIGIDO! 🇧🇷 ===${RESET}\n\nDigite apenas:\n\nbaixar\n"

```
📱 Como abrir o programa depois de instalado?
Sempre que abrir o Termux, basta digitar:
```bash
baixar
