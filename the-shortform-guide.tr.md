# Claude Code'a Dair Her Şey İçin Kısa Rehber

![Başlık: Anthropic Hackathon Kazananı - Claude Code İçin İpuçları ve Püf Noktaları](./assets/images/shortform/00-header.png)

---

**Şubat ayındaki deneysel kullanıma açılışından beri sıkı bir Claude Code kullanıcısıyım; ayrıca [@DRodriguezFX](https://x.com/DRodriguezFX) ile birlikte [zenith.chat](https://zenith.chat)'i tamamen Claude Code kullanarak geliştirip Anthropic x Forum Ventures hackathon'unu kazandık.**

10 aylık günlük kullanımın ardından eksiksiz kurulumum burada: skills, hooks, subagents, MCP'ler, plugins ve gerçekten işe yarayanlar.

---

## Skills ve Commands

Skills, temel iş akışı yüzeyidir. Belirli bir yürütme desenine ihtiyaç duyduğunuzda yeniden kullanılabilir prompt'ları, yapıyı, destek dosyalarını ve codemap'leri bir araya getiren kapsamlandırılmış iş akışı paketleri gibi çalışırlar.

Opus 4.5 ile uzun bir kodlama seansından sonra ölü kodları ve ortalıkta kalmış `.md` dosyalarını temizlemek mi istiyorsunuz? `/refactor-clean` çalıştırın. Test mi gerekiyor? `/tdd`, `/e2e`, `/test-coverage`. Bu slash girişleri kullanışlıdır; fakat kalıcı ve dayanıklı asıl birim, alttaki skill'dir. Skills ayrıca Claude'un keşif sırasında bağlam harcamadan kod tabanınızda hızlıca gezinmesini sağlayan codemap'ler de içerebilir.

![Zincirlenmiş komutları gösteren terminal](./assets/images/shortform/02-chaining-commands.jpeg)
*Komutları birbirine zincirlemek*

ECC hâlâ bir `commands/` katmanı ile gelir; ancak bunu geçiş sürecindeki eski slash-entry uyumluluğu olarak düşünmek en doğrusudur. Kalıcı mantık skills içinde yaşamalıdır.

- **Skills**: `~/.claude/skills/` - kanonik iş akışı tanımları
- **Commands**: `~/.claude/commands/` - hâlâ ihtiyaç duyduğunuzda kullanılan eski slash-entry shim'leri

```bash
# Örnek skill yapısı
~/.claude/skills/
  pmx-guidelines.md      # Projeye özel desenler
  coding-standards.md    # Dil için en iyi uygulamalar
  tdd-workflow/          # SKILL.md içeren çok dosyalı skill
  security-review/       # Kontrol listesi tabanlı skill
```

---

## Hooks

Hooks, belirli olaylarda tetiklenen otomasyonlardır. Skills'ten farklı olarak araç çağrıları ve yaşam döngüsü olaylarıyla sınırlıdırlar.

**Hook Türleri:**

1. **PreToolUse** - Bir araç çalışmadan önce (doğrulama, hatırlatmalar)
2. **PostToolUse** - Bir araç çalışmayı bitirdikten sonra (formatlama, geri bildirim döngüleri)
3. **UserPromptSubmit** - Mesaj gönderdiğinizde
4. **Stop** - Claude yanıt vermeyi bitirdiğinde
5. **PreCompact** - Bağlam sıkıştırılmadan önce
6. **Notification** - İzin istekleri

**Örnek: uzun süren komutlardan önce tmux hatırlatıcısı**

```json
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] Consider tmux for session persistence' >&2; fi"
        }
      ]
    }
  ]
}
```

![PostToolUse hook geri bildirimi](./assets/images/shortform/03-posttooluse-hook.png)
*Claude Code'da bir PostToolUse hook'u çalıştırırken aldığınız geri bildirime örnek*

**Profesyonel ipucu:** Hook'ları JSON'u elle yazmak yerine konuşarak oluşturmak için `hookify` plugin'ini kullanın. `/hookify` çalıştırın ve ne istediğinizi anlatın.

---

## Subagents

Subagents, orkestratörünüzün (ana Claude'un) sınırlı kapsamlarla görev devredebileceği süreçlerdir. Arka planda veya ön planda çalışabilirler; böylece ana ajan için bağlam boşaltırlar.

Subagents, skills ile çok iyi çalışır: Skills'inizin bir alt kümesini yürütebilen bir subagent'e görev devredilebilir ve o subagent bu skills'i otonom şekilde kullanabilir. Ayrıca belirli araç izinleriyle sandbox içine de alınabilirler.

```bash
# Örnek subagent yapısı
~/.claude/agents/
  planner.md           # Özellik uygulama planlaması
  architect.md         # Sistem tasarımı kararları
  tdd-guide.md         # Test odaklı geliştirme
  code-reviewer.md     # Kalite/güvenlik incelemesi
  security-reviewer.md # Zafiyet analizi
  build-error-resolver.md
  e2e-runner.md
  refactor-cleaner.md
```

Doğru kapsamlandırma için izin verilen araçları, MCP'leri ve izinleri her subagent bazında yapılandırın.

---

## Kurallar ve Bellek

`.rules` klasörünüz, Claude'un HER ZAMAN takip etmesi gereken en iyi uygulamaları içeren `.md` dosyalarını barındırır. İki yaklaşım vardır:

1. **Tek CLAUDE.md** - Her şeyi tek dosyada tutmak (kullanıcı veya proje seviyesinde)
2. **Rules klasörü** - İlgi alanına göre gruplanmış modüler `.md` dosyaları

```bash
~/.claude/rules/
  security.md      # Hardcoded secret yok, girdileri doğrula
  coding-style.md  # Değişmezlik, dosya organizasyonu
  testing.md       # TDD iş akışı, %80 kapsam
  git-workflow.md  # Commit formatı, PR süreci
  agents.md        # Ne zaman subagent'e devredileceği
  performance.md   # Model seçimi, bağlam yönetimi
```

**Örnek kurallar:**

- Kod tabanında emoji kullanma
- Frontend'de mor tonlardan kaçın
- Dağıtımdan önce kodu her zaman test et
- Mega dosyalar yerine modüler kodu önceliklendir
- `console.log` commit'leme

---

## MCP'ler (Model Context Protocol)

MCP'ler Claude'u doğrudan harici servislere bağlar. API'lerin yerine geçmezler; daha esnek bilgi gezintisi sağlayan, prompt odaklı bir sarmalayıcıdırlar.

**Örnek:** Supabase MCP, Claude'un belirli verileri çekmesini ve copy-paste yapmadan doğrudan upstream SQL çalıştırmasını sağlar. Aynı mantık veritabanları, dağıtım platformları vb. için de geçerlidir.

![Tabloları listeleyen Supabase MCP](./assets/images/shortform/04-supabase-mcp.jpeg)
*Supabase MCP'nin public schema içindeki tabloları listelemesine örnek*

**Claude içinde Chrome:** Claude'un tarayıcınızı otonom şekilde kontrol etmesini sağlayan yerleşik bir plugin MCP'dir; nasıl çalıştığını görmek için tıklamalar yapabilir.

**KRİTİK: Bağlam Penceresi Yönetimi**

MCP seçiminde seçici olun. Ben tüm MCP'leri kullanıcı yapılandırmasında tutuyorum ama **kullanmadığım her şeyi devre dışı bırakıyorum**. `/plugins`'e gidip aşağı kaydırın veya `/mcp` çalıştırın.

![/plugins arayüzü](./assets/images/shortform/05-plugins-interface.jpeg)
*Şu anda kurulu MCP'leri ve durumlarını görmek için `/plugins` ile MCP'lere gitmek*

Sıkıştırma öncesi 200k'lık bağlam pencereniz, çok fazla araç etkin olduğunda fiilen 70k'ya düşebilir. Performans ciddi biçimde geriler.

**Pratik kural:** Yapılandırmada 20-30 MCP olabilir; ancak etkin MCP sayısını 10'un altında, aktif araç sayısını da 80'in altında tutun.

```bash
# Etkin MCP'leri kontrol et
/mcp

# Kullanılmayanları ~/.claude/settings.json içinde veya mevcut repo'nun .mcp.json dosyasında devre dışı bırak
```

---

## Plugins

Plugins, zahmetli manuel kurulum yerine araçları kolay kurulum için paketler. Bir plugin, skill + MCP kombinasyonu olabilir ya da hooks/tools birlikte paketlenmiş olabilir.

**Plugin kurulumu:**

```bash
# Bir marketplace ekle
# @mixedbread-ai tarafından mgrep plugin'i
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# Claude'u aç, /plugins çalıştır, yeni marketplace'i bul ve oradan kur
```

![mgrep gösteren Marketplaces sekmesi](./assets/images/shortform/06-marketplaces-mgrep.jpeg)
*Yeni kurulan Mixedbread-Grep marketplace'inin gösterimi*

**LSP Plugins**, Claude Code'u sık sık editör dışında çalıştırıyorsanız özellikle faydalıdır. Language Server Protocol, bir IDE açık olmadan Claude'a gerçek zamanlı tip kontrolü, tanıma gitme ve akıllı tamamlama sağlar.

```bash
# Etkin plugin örneği
typescript-lsp@claude-plugins-official  # TypeScript zekâsı
pyright-lsp@claude-plugins-official     # Python tip kontrolü
hookify@claude-plugins-official         # Hook'ları konuşarak oluşturma
mgrep@Mixedbread-Grep                   # ripgrep'ten daha iyi arama
```

MCP'lerdeki aynı uyarı burada da geçerli: bağlam pencerenizi izleyin.

---

## İpuçları ve Püf Noktaları

### Klavye Kısayolları

- `Ctrl+U` - Tüm satırı siler (backspace'e basıp durmaktan hızlı)
- `!` - Hızlı bash komutu ön eki
- `@` - Dosya arama
- `/` - Slash komutlarını başlatma
- `Shift+Enter` - Çok satırlı giriş
- `Tab` - Thinking görünümünü aç/kapat
- `Esc Esc` - Claude'u kesintiye uğrat / kodu geri yükle

### Paralel İş Akışları

- **Fork** (`/fork`) - Kuyruğa mesaj yığmak yerine çakışmayan görevleri paralel yürütmek için konuşmaları fork'lar
- **Git Worktrees** - Çakışma olmadan örtüşen paralel Claude oturumları için. Her worktree bağımsız bir checkout'tur

```bash
git worktree add ../feature-branch feature-branch
# Artık her worktree içinde ayrı Claude instance'ları çalıştırın
```

### Uzun Süren Komutlar İçin tmux

Claude'un çalıştırdığı log/bash süreçlerini stream edin ve izleyin:

[İzle: uzun süren bir komutu stream eden tmux oturumu (video)](./assets/images/shortform/07-tmux-video.mp4)

```bash
tmux new -s dev
# Claude komutları burada çalıştırır; siz detach edip tekrar bağlanabilirsiniz
tmux attach -t dev
```

### mgrep > grep

`mgrep`, ripgrep/grep'e göre ciddi bir iyileştirmedir. Plugin marketplace üzerinden kurun, ardından `/mgrep` skill'ini kullanın. Hem yerel arama hem de web aramasıyla çalışır.

```bash
mgrep "function handleSubmit"  # Yerel arama
mgrep --web "Next.js 15 app router changes"  # Web araması
```

### Diğer Faydalı Komutlar

- `/rewind` - Önceki bir duruma geri dön
- `/statusline` - Branch, bağlam %, todo'lar ile özelleştir
- `/checkpoints` - Dosya seviyesinde geri alma noktaları
- `/compact` - Bağlam sıkıştırmayı manuel tetikle

### GitHub Actions CI/CD

GitHub Actions ile PR'larınızda kod incelemesi kurun. Doğru yapılandırıldığında Claude PR'ları otomatik inceleyebilir.

![Bir PR'ı onaylayan Claude bot](./assets/images/shortform/08-github-pr-review.jpeg)
*Claude'un bir bug fix PR'ını onaylaması*

### Sandboxing

Riskli işlemler için sandbox modunu kullanın: Claude, gerçek sisteminizi etkilemeden kısıtlı bir ortamda çalışır.

---

## Editörler Üzerine

Editör tercihiniz Claude Code iş akışını ciddi şekilde etkiler. Claude Code herhangi bir terminalden çalışsa da onu yetenekli bir editörle eşleştirmek gerçek zamanlı dosya takibi, hızlı gezinme ve entegre komut yürütme sağlar.

### Zed (Benim Tercihim)

Ben [Zed](https://zed.dev) kullanıyorum. Rust ile yazıldığı için gerçekten hızlı. Anında açılıyor, dev kod tabanlarını zorlanmadan kaldırıyor ve sistem kaynaklarına neredeyse dokunmuyor.

**Zed + Claude Code neden harika bir kombinasyon:**

- **Hız** - Rust tabanlı performans, Claude dosyaları hızla düzenlerken gecikme olmaması demektir. Editörünüz tempoya ayak uydurur
- **Agent Panel Entegrasyonu** - Zed'in Claude entegrasyonu, Claude dosyaları düzenledikçe değişiklikleri gerçek zamanlı izlemenizi sağlar. Editörden ayrılmadan Claude'un referans verdiği dosyalar arasında geçebilirsiniz
- **CMD+Shift+R Command Palette** - Tüm özel slash komutlarınıza, debugger'lara ve build script'lerine aranabilir bir arayüzden hızlı erişim
- **Minimum Kaynak Kullanımı** - Ağır işlemler sırasında RAM/CPU için Claude ile yarışmaz. Opus çalıştırırken önemlidir
- **Vim Mode** - Böyle seviyorsanız tam Vim keybinding desteği

![Özel komutlarla Zed Editor](./assets/images/shortform/09-zed-editor.jpeg)
*CMD+Shift+R ile özel komut açılır menüsü gösterilen Zed Editor. Sağ altta bullseye simgesiyle Following mode görünüyor.*

**Editörden Bağımsız İpuçları:**

1. **Ekranı bölün** - Bir tarafta Claude Code terminali, diğer tarafta editör
2. **Ctrl + G** - Claude'un o anda Zed'de üzerinde çalıştığı dosyayı hızlı açar
3. **Auto-save** - Claude'un dosya okumaları her zaman güncel olsun diye otomatik kaydetmeyi etkinleştirin
4. **Git entegrasyonu** - Commit etmeden önce Claude'un değişikliklerini incelemek için editörünüzün git özelliklerini kullanın
5. **File watchers** - Çoğu editör değişen dosyaları otomatik yeniden yükler; bunun etkin olduğundan emin olun

### VSCode / Cursor

Bu da geçerli bir seçenek ve Claude Code ile iyi çalışır. Terminal formatında kullanabilir, `\ide` ile LSP işlevselliğini etkinleştirerek editörünüzle otomatik senkronizasyon sağlayabilirsiniz (plugin'ler nedeniyle artık bir miktar tekrarlı). Ya da editörle daha entegre ve uyumlu bir arayüze sahip extension'ı tercih edebilirsiniz.

![VS Code Claude Code Extension](./assets/images/shortform/10-vscode-extension.jpeg)
*VS Code extension'ı, Claude Code için IDE'ye doğrudan entegre edilmiş yerel bir grafik arayüz sunar.*

---

## Benim Kurulumum

### Plugins

**Kurulu:** (Genelde bunlardan aynı anda yalnızca 4-5 tanesini etkin tutuyorum)

```markdown
ralph-wiggum@claude-code-plugins       # Döngü otomasyonu
frontend-patterns@claude-code-plugins  # UI/UX desenleri
commit-commands@claude-code-plugins    # Git iş akışı
security-guidance@claude-code-plugins  # Güvenlik kontrolleri
pr-review-toolkit@claude-code-plugins  # PR otomasyonu
typescript-lsp@claude-plugins-official # TS zekâsı
hookify@claude-plugins-official        # Hook oluşturma
code-simplifier@claude-plugins-official
feature-dev@claude-code-plugins
explanatory-output-style@claude-code-plugins
code-review@claude-code-plugins
context7@claude-plugins-official       # Canlı dokümantasyon
pyright-lsp@claude-plugins-official    # Python tipleri
mgrep@Mixedbread-Grep                  # Daha iyi arama
```

### MCP Sunucuları

**Yapılandırılmış (Kullanıcı Seviyesi):**

```json
{
  "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
  "firecrawl": { "command": "npx", "args": ["-y", "firecrawl-mcp"] },
  "supabase": {
    "command": "npx",
    "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_REF"]
  },
  "memory": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-memory"] },
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  },
  "vercel": { "type": "http", "url": "https://mcp.vercel.com" },
  "railway": { "command": "npx", "args": ["-y", "@railway/mcp-server"] },
  "cloudflare-docs": { "type": "http", "url": "https://docs.mcp.cloudflare.com/mcp" },
  "cloudflare-workers-bindings": {
    "type": "http",
    "url": "https://bindings.mcp.cloudflare.com/mcp"
  },
  "clickhouse": { "type": "http", "url": "https://mcp.clickhouse.cloud/mcp" },
  "AbletonMCP": { "command": "uvx", "args": ["ableton-mcp"] },
  "magic": { "command": "npx", "args": ["-y", "@magicuidesign/mcp@latest"] }
}
```

Asıl nokta bu: 14 MCP yapılandırılmış durumda ama proje başına yalnızca yaklaşık 5-6 tanesi etkin. Bu, bağlam penceresini sağlıklı tutuyor.

### Temel Hooks

```json
{
  "PreToolUse": [
    { "matcher": "npm|pnpm|yarn|cargo|pytest", "hooks": ["tmux reminder"] },
    { "matcher": "Write && .md file", "hooks": ["block unless README/CLAUDE"] },
    { "matcher": "git push", "hooks": ["open editor for review"] }
  ],
  "PostToolUse": [
    { "matcher": "Edit && .ts/.tsx/.js/.jsx", "hooks": ["prettier --write"] },
    { "matcher": "Edit && .ts/.tsx", "hooks": ["tsc --noEmit"] },
    { "matcher": "Edit", "hooks": ["grep console.log warning"] }
  ],
  "Stop": [
    { "matcher": "*", "hooks": ["check modified files for console.log"] }
  ]
}
```

### Özel Status Line

Kullanıcıyı, dizini, kirli durum göstergeli git branch'ini, kalan bağlam yüzdesini, modeli, saati ve todo sayısını gösterir:

![Özel status line](./assets/images/shortform/11-statusline.jpeg)
*Mac root dizinimdeki statusline örneği*

```
affoon:~ ctx:65% Opus 4.5 19:52
▌▌ plan mode on (shift+tab to cycle)
```

### Rules Yapısı

```
~/.claude/rules/
  security.md      # Zorunlu güvenlik kontrolleri
  coding-style.md  # Değişmezlik, dosya boyutu sınırları
  testing.md       # TDD, %80 kapsam
  git-workflow.md  # Conventional commits
  agents.md        # Subagent delegasyon kuralları
  patterns.md      # API response formatları
  performance.md   # Model seçimi (Haiku vs Sonnet vs Opus)
  hooks.md         # Hook dokümantasyonu
```

### Subagents

```
~/.claude/agents/
  planner.md           # Özellikleri parçalara ayırır
  architect.md         # Sistem tasarımı
  tdd-guide.md         # Önce test yazdırır
  code-reviewer.md     # Kalite incelemesi
  security-reviewer.md # Zafiyet taraması
  build-error-resolver.md
  e2e-runner.md        # Playwright testleri
  refactor-cleaner.md  # Ölü kod temizliği
  doc-updater.md       # Dokümanları senkron tutar
```

---

## Temel Çıkarımlar

1. **Aşırı karmaşıklaştırmayın** - yapılandırmayı mimari gibi değil, fine-tuning gibi ele alın
2. **Bağlam penceresi değerlidir** - kullanılmayan MCP'leri ve plugin'leri devre dışı bırakın
3. **Paralel yürütme** - konuşmaları fork'layın, git worktrees kullanın
4. **Tekrarlayan işleri otomatikleştirin** - formatlama, lint, hatırlatmalar için hooks
5. **Subagents kapsamını dar tutun** - sınırlı araçlar = odaklı yürütme

---

## Referanslar

- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference)
- [Hooks Documentation](https://code.claude.com/docs/en/hooks)
- [Checkpointing](https://code.claude.com/docs/en/checkpointing)
- [Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Memory System](https://code.claude.com/docs/en/memory)
- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [MCP Overview](https://code.claude.com/docs/en/mcp-overview)

---

**Not:** Bu, ayrıntıların bir alt kümesidir. Gelişmiş desenler için [Longform Guide](./the-longform-guide.md)'a bakın.

---

*[zenith.chat](https://zenith.chat)'i [@DRodriguezFX](https://x.com/DRodriguezFX) ile birlikte geliştirerek NYC'deki Anthropic x Forum Ventures hackathon'unu kazandık.*
