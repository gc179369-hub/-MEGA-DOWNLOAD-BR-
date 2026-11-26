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
export DEBIAN_FRONTEND=noninteractive && pkg update -y -o Dpkg::Options::="--force-confnew" && pkg upgrade -y -o Dpkg::Options::="--force-confnew" && pkg install ffmpeg curl ncurses-utils ca-certificates python python-pip file -y && pip install --upgrade "yt-dlp[default,curl-cffi]" gallery-dl && cat << 'EOF' > "$HOME/baixar_src.sh"
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
if ! mkdir -p "$PASTA_DESTINO" 2>/dev/null || [ ! -w "$PASTA_DESTINO" ]; then
    echo -e "${AMARELO}Sem acesso ao armazenamento. Rode: termux-setup-storage${RESET}"
    PASTA_DESTINO="$HOME/Download"
    mkdir -p "$PASTA_DESTINO"
fi

LOG="$HOME/.baixar_ultimo.log"
RELATORIO="$HOME/relatorio_baixar.txt"
UA="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36"
HDRS_DESKTOP=(-H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
              -H "Accept-Language: pt-BR,pt;q=0.9,en;q=0.8")

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

extrair_url() {
    local toks t
    read -ra toks <<< "$1"
    for t in "${toks[@]}"; do
        case "$t" in http://*|https://*) echo "$t"; return 0;; esac
    done
    return 1
}

normalizar_locale() {
    local url="$1"
    if [[ "$url" =~ ^https?://[a-z]{2}\.((xhamster|pornhub|youporn|redtube|xvideos|xnxx)\.[a-z.]+)(/.*)?$ ]]; then
        echo "https://www.${BASH_REMATCH[1]}${BASH_REMATCH[3]}"
    else
        echo "$url"
    fi
}

resolver_redirect() {
    local url="$1" final
    case "$url" in
        *reddit.com/r/*/s/*|*vm.tiktok.com*|*vt.tiktok.com*|*k.kwai.com*)
            final=$(curl -s -o /dev/null -L --max-time 10 -A "$UA" "${HDRS_DESKTOP[@]}" -w '%{url_effective}' "$url" 2>>"$LOG")
            if [ -n "$final" ] && [ "$final" != "$url" ]; then
                echo "$final"; return 0
            fi
            ;;
    esac
    echo "$url"
}

preparar_url() {
    local url
    url=$(normalizar_locale "$1")
    resolver_redirect "$url"
}

validar_video() {
    local f="$1" info codec width mime
    if command -v file >/dev/null 2>&1; then
        mime=$(file -b --mime-type "$f" 2>/dev/null)
        case "$mime" in
            image/*|text/*|application/json|application/octet-stream) return 1 ;;
        esac
    fi
    if command -v ffprobe >/dev/null 2>&1; then
        info=$(ffprobe -v error -select_streams v:0 -show_entries stream=codec_name,width,height \
               -of csv=p=0 "$f" 2>/dev/null)
        [ -z "$info" ] && return 1
        codec=$(echo "$info" | cut -d, -f1)
        width=$(echo "$info" | cut -d, -f2)
        case "$codec" in
            mjpeg|png|bmp|gif|webp|avif|"") return 1 ;;
        esac
        [ -n "$width" ] && [ "$width" -gt 0 ] 2>/dev/null
        return $?
    fi
    return 0
}

remuxar_mp4_limpo() {
    local src="$1" dst="$2"
    command -v ffmpeg >/dev/null 2>&1 || return 1
    ffmpeg -y -loglevel error -fflags +genpts -i "$src" \
        -map 0 -c copy -avoid_negative_ts make_zero -movflags +faststart \
        -f mp4 "$dst" </dev/null 2>>"$LOG" && [ -s "$dst" ]
}

sanitizar_nome() {
    local s="$1"
    s=$(printf '%s' "$s" | sed -E 's#[/\\:*?"<>|]+#_#g' | sed -E 's/\s+/ /g' | sed -E 's/^ +| +$//g')
    printf '%.65s' "$s"
}

traduzir_para_ptbr() {
    local texto="$1" encoded resp trad
    [ -z "$texto" ] && return 1
    command -v python >/dev/null 2>&1 || { printf '%s' "$texto"; return 0; }

    encoded=$(printf '%s' "$texto" | python -c 'import sys, urllib.parse; print(urllib.parse.quote(sys.stdin.read().strip()))' 2>/dev/null)
    [ -z "$encoded" ] && { printf '%s' "$texto"; return 0; }

    resp=$(curl -fsSL --max-time 8 -A "$UA" "https://api.mymemory.translated.net/get?q=${encoded}&langpair=autodetect|pt-BR" 2>>"$LOG") || {
        printf '%s' "$texto"; return 0
    }

    trad=$(printf '%s' "$resp" | python -c '
import sys, json
try:
    data = json.load(sys.stdin)
    t = data.get("responseData", {}).get("translatedText", "")
    if t and "MYMEMORY WARNING" not in t.upper():
        print(t)
except Exception:
    pass
' 2>/dev/null)

    if [ -n "$trad" ]; then
        printf '%s' "$trad"
    else
        printf '%s' "$texto"
    fi
}

obter_titulo_traduzido() {
    local url="$1" html="$2" titulo trad
    if [ -z "$html" ]; then
        html=$(curl -fsSL --compressed --max-time 20 -A "$UA" "${HDRS_DESKTOP[@]}" "$url" 2>>"$LOG" || true)
    fi

    titulo=$(printf '%s' "$html" | grep -ioE '<meta[^>]+property="og:title"[^>]+content="[^"]*"' | head -n1 | grep -oE 'content="[^"]*"' | sed -E 's/^content="//; s/"$//')
    [ -z "$titulo" ] && titulo=$(printf '%s' "$html" | grep -ioE '<title[^>]*>[^<]*</title>' | head -n1 | sed -E 's/<[^>]*>//g')
    titulo=$(printf '%s' "$titulo" | sed -e 's/&amp;/\&/g' -e 's/&quot;/"/g' -e "s/&#039;/'/g" -e 's/ - Pornhub.*//i' -e 's/ \| Pornhub.*//i')

    if [ -z "$titulo" ] && command -v yt-dlp >/dev/null 2>&1; then
        titulo=$(yt-dlp --no-warnings --impersonate chrome --skip-download --print "%(title)s" "$url" 2>>"$LOG" | head -n1 || true)
    fi

    [ -z "$titulo" ] && return 1
    trad=$(traduzir_para_ptbr "$titulo")
    sanitizar_nome "$trad"
}

filtrar_e_ordenar_videos() {
    grep -vE 'videos_screenshots|screenshots|_preview\.mp4|/preview[^a-zA-Z0-9_-]|preview\.mp4|\.(jpg|jpeg|png|webp|avif|gif)(\?|$)|format=(avif|webp|jpg|jpeg|png)' \
    | awk '!seen[$0]++' \
    | awk '
    {
        line = $0
        tmp = line
        max_num = 0
        while (match(tmp, /[0-9]{3,4}[pP]/)) {
            n = substr(tmp, RSTART, RLENGTH - 1) + 0
            if (n > max_num) max_num = n
            tmp = substr(tmp, RSTART + RLENGTH)
        }
        if (max_num == 0 && match(line, /[_-][0-9]{3,4}\.(mp4|m3u8|webm)/)) {
            sub_str = substr(line, RSTART, RLENGTH)
            match(sub_str, /[0-9]{3,4}/)
            if (RSTART > 0) max_num = substr(sub_str, RSTART, RLENGTH) + 0
        }

        has_res = (max_num > 0) ? 1 : 0
        group = (has_res == 1) ? 0 : 1
        inv_res = 999999 - max_num
        label = (has_res == 1) ? (max_num "p") : "sem-res"
        printf "%d\t%06d\t%s\t%d\t%s\n", group, inv_res, label, has_res, line
    }' | sort -t$'\t' -k1,1n -k2,2n | cut -f3-
}

entregar() {
    local dir="$1" modo="$2" titulo="$3" f base destino n=0 ext remux_tmp
    while IFS= read -r -d '' f; do
        base=$(basename "$f")
        if [ "$modo" = "audio" ]; then
            if [ -n "$titulo" ]; then destino="$PASTA_DESTINO/$titulo.mp3"; else destino="$PASTA_DESTINO/${base%.*}.mp3"; fi
            [ -f "$destino" ] && destino="${destino%.mp3}-$RANDOM.mp3"
            ffmpeg -y -loglevel error -i "$f" -vn -codec:a libmp3lame -q:a 0 "$destino" </dev/null && n=$((n+1))
        else
            if ! validar_video "$f"; then
                echo "arquivo rejeitado (nao e video real / screenshot): $f" >> "$LOG"
                continue
            fi
            ext="${base##*.}"
            case "$ext" in
                php|part|htm|html|ts|m3u8|""|*[!a-zA-Z0-9]*) ext="mp4" ;;
                jpg|jpeg|png|webp|avif|gif) continue ;;
            esac
            if [ -n "$titulo" ]; then destino="$PASTA_DESTINO/$titulo.$ext"; else destino="$PASTA_DESTINO/video_$RANDOM.$ext"; fi
            [ -f "$destino" ] && destino="${destino%.*}-$RANDOM.$ext"

            remux_tmp="${dir}/remux_${RANDOM}.mp4"
            if [ "$ext" = "mp4" ] && remuxar_mp4_limpo "$f" "$remux_tmp" && validar_video "$remux_tmp"; then
                mv -f "$remux_tmp" "$destino" && n=$((n+1))
            else
                rm -f "$remux_tmp"
                mv -f "$f" "$destino" && n=$((n+1))
            fi
        fi
    done < <(find "$dir" -type f -size +50k -print0)
    [ "$n" -gt 0 ]
}

baixar_capa_png() {
    local html="$1" titulo="$2" png_url destino
    [ -z "$titulo" ] && titulo="capa_$RANDOM"
    png_url=$(printf '%s' "$html" \
        | sed -e 's#\\/#/#g' -e 's#\\u0026#\&#g' -e 's#&amp;#\&#g' \
        | grep -oE 'https?://[^"'\'' <>\\)]+\.(png|jpg|jpeg)[^"'\'' <>\\)]*' \
        | head -n1)
    [ -z "$png_url" ] && return 1
    destino="$PASTA_DESTINO/${titulo}_capa.png"
    [ -f "$destino" ] && destino="$PASTA_DESTINO/${titulo}_capa-$RANDOM.png"
    if curl -fsSL --max-time 30 -A "$UA" "$png_url" -o "$destino" 2>>"$LOG" && [ -s "$destino" ]; then
        echo -e "${AMARELO}🖼 Capa salva em: $destino${RESET}"
        return 0
    fi
    rm -f "$destino"
    return 1
}

tentar_pimpbunny() {
    local url="$1" modo="$2" interativo="${3:-sim}" html titulo todos_cand com_res sem_res lista_final res has_res u tmp resp_capa idx=0
    echo -e "${CIANO}-> varrendo página e listando MP4s encontrados...${RESET}"
    html=$(curl -fsSL --compressed --max-time 30 -A "$UA" "${HDRS_DESKTOP[@]}" -e "$url" "$url" 2>>"$LOG") || return 1
    titulo=$(obter_titulo_traduzido "$url" "$html")
    [ -n "$titulo" ] && echo -e "${CIANO}-> título (PT-BR): ${titulo}${RESET}"
    [ -z "$titulo" ] && titulo="pimpbunny_$RANDOM"

    todos_cand=$(printf '%s' "$html" \
        | sed -e 's#\\/#/#g' -e 's#\\u0026#\&#g' -e 's#&amp;#\&#g' \
        | grep -oE 'https?://[^"'\'' <>\\)]*(get_file/[^"'\'' <>\\)]*\.mp4[^"'\'' <>\\)]*|\.(mp4|m3u8)(\?[^"'\'' <>\\)]*)?)' \
        | filtrar_e_ordenar_videos)

    if [ -n "$todos_cand" ]; then
        echo -e "${CIANO}--- RESULTADOS DA VARREDURA ---${RESET}"
        while IFS=$'\t' read -r res has_res u; do
            [ -z "$u" ] && continue
            idx=$((idx+1))
            echo -e "${AMARELO}  [$idx] resolução: ${res} -> ${u:0:90}...${RESET}"
        done <<< "$todos_cand"
        echo -e "${CIANO}--------------------------------${RESET}"

        com_res=$(printf '%s\n' "$todos_cand" | awk -F'\t' '$2 == 1 {print $0}')
        if [ -n "$com_res" ]; then
            echo -e "${VERDE}✔ Priorizando links com resolução explícita na URL (maior primeiro)...${RESET}"
            lista_final="$com_res"
        else
            sem_res=$(printf '%s\n' "$todos_cand" | awk -F'\t' '$2 == 0 {print $0}')
            echo -e "${AMARELO}⚠ Nenhum link com resolução na URL. Tentando links base [sem-res]...${RESET}"
            lista_final="$sem_res"
        fi

        while IFS=$'\t' read -r res has_res u; do
            [ -z "$u" ] && continue
            echo -e "${AMARELO}  baixando direto do servidor [$res]: $u${RESET}"
            tmp=$(mktemp -d)
            if yt-dlp -P "$tmp" -o "midia_curta.mp4" --windows-filenames --no-playlist \
                  --compat-options allow-unsafe-ext --impersonate chrome \
                  --add-header "Referer:$url" --user-agent "$UA" "$u" \
                  </dev/null 2> >(tee -a "$LOG" >&2) && entregar "$tmp" "$modo" "$titulo"; then
                rm -rf "$tmp"
                return 0
            fi
            if curl -fL --progress-bar --max-time 600 -A "$UA" -e "$url" "$u" -o "$tmp/direto.mp4" 2>>"$LOG" \
               && entregar "$tmp" "$modo" "$titulo"; then
                rm -rf "$tmp"
                return 0
            fi
            rm -rf "$tmp"
        done <<< "$lista_final"
    fi

    echo -e "${AMARELO}⚠ Nenhum MP4 válido encontrado ou baixado no HTML deste vídeo.${RESET}"
    if [ "$interativo" = "sim" ]; then
        read -p "Deseja baixar apenas a capa se disponível? (s/N): " resp_capa
        case "$(echo "$resp_capa" | tr '[:upper:]' '[:lower:]')" in
            s|sim) baixar_capa_png "$html" "$titulo" ;;
        esac
    fi
    return 1
}

tentar_gallerydl() {
    local url="$1" modo="$2" tmp rc titulo
    command -v gallery-dl >/dev/null 2>&1 || return 1
    echo -e "${CIANO}-> fallback: gallery-dl${RESET}"
    titulo=$(obter_titulo_traduzido "$url" "" || true)
    tmp=$(mktemp -d)
    gallery-dl -q -d "$tmp" "$url" </dev/null 2> >(tee -a "$LOG" >&2)
    entregar "$tmp" "$modo" "$titulo"; rc=$?
    rm -rf "$tmp"; return $rc
}

extrair_midias_html() {
    local PAT='https?://[^"'"'"' <>\\)]+\.(mp4|m3u8|webm|mov)(\?[^"'"'"' <>\\)]*)?'
    local html
    html=$(sed -e 's#\\/#/#g' -e 's#\\u0026#\&#g' -e 's#&amp;#\&#g')
    {
        printf '%s' "$html" | grep -oE 'og:video[^>]*content="[^"]+"' | grep -oE 'https?://[^"]+'
        printf '%s' "$html" | grep -oE 'og:video:secure_url"[^>]*content="[^"]+"' | grep -oE 'https?://[^"]+'
        printf '%s' "$html" | grep -oE '"contentUrl"\s*:\s*"[^"]+"' | grep -oE 'https?://[^"]+'
        printf '%s' "$html" | grep -oE "$PAT"
    } | filtrar_e_ordenar_videos | awk -F'\t' '{print $3}' | head -n 8
}

tentar_scraper() {
    local url="$1" modo="$2" html found u tmp titulo
    echo -e "${CIANO}-> fallback: busca direta no HTML da página (modo PC)${RESET}"
    html=$(curl -fsSL --compressed --max-time 30 -A "$UA" "${HDRS_DESKTOP[@]}" -e "$url" "$url" 2>>"$LOG") || return 1
    titulo=$(obter_titulo_traduzido "$url" "$html")
    found=$(printf '%s' "$html" | extrair_midias_html)
    [ -z "$found" ] && { echo "nenhum link de midia achado no HTML (provavel pagina renderizada por JavaScript)" >> "$LOG"; return 1; }
    while IFS= read -r u; do
        [ -z "$u" ] && continue
        echo -e "${AMARELO}  candidato: $u${RESET}"
        tmp=$(mktemp -d)
        if yt-dlp -P "$tmp" -o "midia_curta.mp4" --windows-filenames --no-playlist \
              --compat-options allow-unsafe-ext --impersonate chrome \
              --add-header "Referer:$url" --user-agent "$UA" "$u" \
              </dev/null 2> >(tee -a "$LOG" >&2) && entregar "$tmp" "$modo" "$titulo"; then
            rm -rf "$tmp"; return 0
        fi
        rm -rf "$tmp"
    done <<< "$found"
    return 1
}

baixar_url() {
    local url_orig="$1" modo="$2" interativo="${3:-sim}" extra n=0 tmp titulo_ptbr
    local -a base modo_args ex
    local url; url=$(preparar_url "$url_orig")

    if [[ "$url" == *rule34video.com* ]]; then
        echo -e "\n${VERMELHO}❌ Site não suportado, digite outro site.${RESET}"
        echo "site nao suportado (rule34video): $url_orig" >> "$LOG"
        return 1
    fi

    : > "$LOG"

    if [[ "$url" == *pimpbunny.com* ]]; then
        tentar_pimpbunny "$url" "$modo" "$interativo" && return 0
    fi

    titulo_ptbr=$(obter_titulo_traduzido "$url" "" || true)
    [ -n "$titulo_ptbr" ] && echo -e "${CIANO}-> título (PT-BR): ${titulo_ptbr}${RESET}"

    if [ "$modo" = "video" ]; then
        if [[ "$url" == *pornhub.com* ]]; then
            echo -e "${CIANO}-> Pornhub detectado: limitando resolução máxima a 1080p...${RESET}"
            modo_args=(-f "bv*[height<=1080][ext=mp4]+ba[ext=m4a]/b[height<=1080][ext=mp4]/bv*[height<=1080]+ba/b[height<=1080]/b" --merge-output-format mp4 --downloader ffmpeg)
        else
            modo_args=(-f "bv*[ext=mp4]+ba[ext=m4a]/bv*+ba/b" --merge-output-format mp4)
        fi
    else
        modo_args=(-x --audio-format mp3 --audio-quality 0)
    fi

    for extra in "--impersonate chrome" "--impersonate chrome --extractor-args generic:impersonate" ""; do
        n=$((n+1))
        echo -e "${CIANO}-> yt-dlp, tentativa $n/3${RESET}"
        ex=(); [ -n "$extra" ] && read -ra ex <<< "$extra"
        tmp=$(mktemp -d)
        base=(-P "$tmp" -o "item_${n}_%(autonumber)s.%(ext)s" --windows-filenames --compat-options allow-unsafe-ext
              --retries 5 --fragment-retries 5 --no-mtime --ignore-errors --user-agent "$UA")
        if yt-dlp "${base[@]}" "${modo_args[@]}" "${ex[@]}" "$url" \
            </dev/null 2> >(tee -a "$LOG" >&2) && entregar "$tmp" "$modo" "$titulo_ptbr"; then
            rm -rf "$tmp"; return 0
        fi
        rm -rf "$tmp"
    done
    tentar_gallerydl "$url" "$modo" && return 0
    tentar_scraper "$url" "$modo" && return 0
    return 1
}

pedir_arquivo() {
    local ARQ
    read -p "Arquivo com os links [$HOME/links.txt]: " ARQ
    ARQ="${ARQ:-$HOME/links.txt}"
    if [ ! -f "$ARQ" ]; then
        echo -e "${VERMELHO}❌ Arquivo não encontrado: $ARQ${RESET}" >&2; return 1
    fi
    echo "$ARQ"
}

modo_lote() {
    local ARQ OP M linha u ok=0 falha=0
    ARQ=$(pedir_arquivo) || { sleep 2; return; }
    read -p "1 = VÍDEO, 2 = ÁUDIO MP3: " OP
    case $OP in 1) M=video;; 2) M=audio;; *) echo -e "${VERMELHO}Opção inválida${RESET}"; sleep 2; return;; esac
    : > "$RELATORIO"
    while IFS= read -r linha || [ -n "$linha" ]; do
        u=$(extrair_url "$linha") || continue
        echo -e "\n${AMARELO}==> $u${RESET}"
        if baixar_url "$u" "$M" "nao"; then
            ok=$((ok+1)); echo "OK    $u" >> "$RELATORIO"
        else
            falha=$((falha+1)); echo "FALHA $u" >> "$RELATORIO"
            grep -E "ERROR|WARNING|nenhum link|site nao suportado|rejeitado" "$LOG" | tail -n 4 | sed 's/^/      /' >> "$RELATORIO"
        fi
    done < "$ARQ"
    matrix_effect
    echo -e "\n${VERDE}✔ LOTE FINALIZADO: $ok ok, $falha falha(s)${RESET}"
    echo -e "${AMARELO}Relatório: cat $RELATORIO${RESET}"
}

modo_testar() {
    local ARQ linha u r erro errfile
    ARQ=$(pedir_arquivo) || { sleep 2; return; }
    : > "$RELATORIO"
    errfile=$(mktemp)
    while IFS= read -r linha || [ -n "$linha" ]; do
        u=$(extrair_url "$linha") || continue
        u=$(preparar_url "$u")
        echo -e "${AMARELO}testando: $u${RESET}"
        if [[ "$u" == *rule34video.com* ]]; then
            echo -e "${VERMELHO}  Site não suportado, digite outro site.${RESET}"
            echo "NAO-SUPORTADO $u :: rule34video" >> "$RELATORIO"
            continue
        fi
        : > "$errfile"
        r=$(yt-dlp --simulate --no-warnings --impersonate chrome --playlist-items 1 --print "%(extractor)s" "$u" 2>"$errfile" </dev/null | head -n1)
        erro=$(tail -n1 "$errfile")
        if [ -n "$r" ]; then
            echo -e "${VERDE}  OK reconhecido ($r)${RESET}"; echo "RECONHECIDO ($r) $u" >> "$RELATORIO"
        else
            echo -e "${VERMELHO}  SEM EXTRATOR: ${erro:-motivo desconhecido}${RESET}"
            echo "SEM-EXTRATOR $u :: ${erro:-sem mensagem de erro}" >> "$RELATORIO"
        fi
    done < "$ARQ"
    rm -f "$errfile"
    echo -e "\n${AMARELO}Relatório: cat $RELATORIO${RESET}"
}

configurar_rule34() {
    local cfg="$HOME/.config/gallery-dl/config.json" uid key
    read -p "rule34.xxx user-id: " uid
    read -p "rule34.xxx api-key: " key
    mkdir -p "$(dirname "$cfg")"
    [ -f "$cfg" ] && cp "$cfg" "$cfg.bak"
    printf '{"extractor":{"rule34":{"user-id":"%s","api-key":"%s"}}}\n' "$uid" "$key" > "$cfg"
    echo -e "${VERDE}✔ Salvo em $cfg${RESET}"; sleep 2
}

while true; do
    clear
    echo -e "${CIANO}==================================================${RESET}"
    echo -e "${VERDE}[BR] ${VERDE_CLARO}🇧🇷  MEGA DOWNLOAD BR  🇧🇷 ${AMARELO}[BR]${RESET}"
    echo -e "${CIANO}==================================================${RESET}"
    echo ""
    echo -e "${AMARELO}👉 DIGITE A PALAVRA-CHAVE [BAIXAR] PARA INICIAR${RESET}"
    echo -e "${AMARELO}👉 DIGITE [LOTE] PARA BAIXAR VÁRIOS LINKS${RESET}"
    echo -e "${AMARELO}👉 DIGITE [TESTAR] PARA TESTAR SUPORTE DOS LINKS${RESET}"
    echo -e "${AMARELO}👉 DIGITE [CONFIGURAR] PARA CHAVES DO RULE34.XXX${RESET}"
    echo -e "${AMARELO}👉 DIGITE [ATUALIZAR] PARA ATUALIZAR O YT-DLP + IMPERSONAÇÃO${RESET}"
    echo -e "${VERMELHO}👉 DIGITE [SAIR] PARA FECHAR O PROGRAMA${RESET}"
    echo ""
    read -p "SISTEMA > " COMANDO

    COMANDO_LOWER=$(echo "$COMANDO" | tr '[:upper:]' '[:lower:]')

    case "$COMANDO_LOWER" in
        sair)
            echo -e "\n${VERDE}Saindo do MEGA DOWNLOAD BR... Até logo!${RESET}\n"
            break
            ;;
        atualizar)
            echo -e "\n${VERDE}⚡ ATUALIZANDO O YT-DLP E MÓDULOS DE IMPERSONAÇÃO...${RESET}"
            pip install --upgrade "yt-dlp[default,curl-cffi]" || pip install --upgrade yt-dlp
            pip install --upgrade gallery-dl
            echo -e "${VERDE}✔ ATUALIZAÇÃO CONCLUÍDA COM SUCESSO!${RESET}"
            sleep 2
            continue
            ;;
        configurar)
            configurar_rule34
            continue
            ;;
        lote)
            modo_lote
            ;;
        testar)
            modo_testar
            ;;
        baixar)
            clear
            echo -e "${CIANO}==================================================${RESET}"
            echo -e "${VERDE}             PAINEL DE CAPTURA HACKER             ${RESET}"
            echo -e "${CIANO}==================================================${RESET}"
            echo ""

            read -p "🔗 COLE A URL DO SEU ALVO AQUI: " URL_RAW
            URL=$(extrair_url "$URL_RAW")

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
            echo -e "${CIANO} [ 1 ] BAIXAR VÍDEO (MÁXIMA QUALIDADE / TODOS OS SITES)${RESET}"
            echo -e "${CIANO} [ 2 ] BAIXAR ÁUDIO (MP3 / EXTRAÇÃO DIRETA NATIVA)     ${RESET}"
            echo -e "${CIANO}==================================================${RESET}"
            read -p "ESCOLHA UMA OPÇÃO (1 OU 2): " OPCAO

            echo ""

            case $OPCAO in
                1)
                    echo -e "${VERDE}⚡ INICIANDO PROTOCOLO DE EXTRAÇÃO DE VÍDEO...${RESET}"
                    if baixar_url "$URL" "video" "sim"; then
                        matrix_effect
                        echo -e "${VERDE}✔ VÍDEO BAIXADO E COMPILADO COM SUCESSO!${RESET}"
                        echo -e "${AMARELO}Salvo em: $PASTA_DESTINO${RESET}"
                    else
                        echo -e "\n${VERMELHO}❌ OCORREU UM ERRO NO PROCESSAMENTO DO VÍDEO ACIMA!${RESET}"
                        echo -e "${AMARELO}Dica: Digite 'atualizar' no menu principal para atualizar as dependências.${RESET}"
                    fi
                    ;;
                2)
                    echo -e "${VERDE}⚡ INICIANDO PROTOCOLO DE EXTRAÇÃO DE ÁUDIO...${RESET}"
                    if baixar_url "$URL" "audio" "sim"; then
                        matrix_effect
                        echo -e "${VERDE}✔ ÁUDIO CONVERTIDO EM MP3 COM SUCESSO!${RESET}"
                        echo -e "${AMARELO}Salvo em: $PASTA_DESTINO${RESET}"
                    else
                        echo -e "\n${VERMELHO}❌ OCORREU UM ERRO NA EXTRAÇÃO DO ÁUDIO ACIMA!${RESET}"
                        echo -e "${AMARELO}Dica: Digite 'atualizar' no menu principal para atualizar as dependências.${RESET}"
                    fi
                    ;;
                *)
                    echo -e "${VERMELHO}❌ OPÇÃO INVÁLIDA NO SISTEMA!${RESET}"
                    ;;
            esac
            ;;
        *)
            echo -e "\n${VERMELHO}❌ PALAVRA-CHAVE INVÁLIDA! USE 'BAIXAR', 'LOTE', 'TESTAR', 'CONFIGURAR', 'ATUALIZAR' OU 'SAIR'.${RESET}"
            sleep 2
            continue
            ;;
    esac

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
done
EOF
chmod +x "$HOME/baixar_src.sh" && mv "$HOME/baixar_src.sh" "$PREFIX/bin/baixar" && clear && printf "=== 🇧🇷 MEGA DOWNLOAD BR (PIMP BUNNY DIRETO + 1080P PORNHUB + TÍTULO PT-BR) ATUALIZADO! 🇧🇷 ===\n\nDigite: baixar\n"
```

📱 Como abrir o programa depois de instalado?
Sempre que abrir o Termux, basta digitar:

```bash
baixar
```