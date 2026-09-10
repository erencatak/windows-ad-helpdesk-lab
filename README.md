# Windows AD Helpdesk Lab

> **EN summary:** A hybrid Active Directory lab built to practice real IT helpdesk / sysadmin troubleshooting. A Domain Controller runs in Azure (Windows Server 2022), and a domain-joined Windows 11 client runs locally in VirtualBox on Apple Silicon — connected over a Tailscale mesh VPN instead of exposing RDP to the public internet. The most valuable part of this repo isn't that everything worked — it's [ERRORS.md](ERRORS.md): 8 real problems I hit, how I diagnosed each one, and what actually fixed it.

---

## 🇹🇷 Bu lab ne kanıtlıyor?

Bu repo, gerçek bir BT destek / sistem yönetimi ortamının küçük ölçekli bir simülasyonu: bulutta bir domain controller, yerelde domain'e katılmış bir istemci, aralarında güvenli (public RDP'siz) bir bağlantı ve bunları kurarken karşılaşılan gerçek hataların adım adım kaydı.

Amaç, "her şey ilk seferde çalıştı" görüntüsü vermek değil — tam tersine, bir sorunla karşılaşınca **nasıl teşhis ettiğimi ve nasıl çözdüğümü** göstermek. Bu yüzden [`ERRORS.md`](ERRORS.md) dosyası bu reponun en önemli parçası.

## Mimari / Topoloji

```mermaid
flowchart LR
    subgraph Azure["☁️ Azure"]
        DC01["DC01\nWindows Server 2022\ncorp.local — Domain Controller\nAD DS + DNS"]
    end
    subgraph VBox["💻 VirtualBox (yerel, Apple Silicon)"]
        CLIENT01["CLIENT01\nWindows 11 ARM64\ncorp.local'a katılmış istemci"]
    end
    subgraph MacHost["🖥️ macOS Host"]
        Host["Tailscale node\n(tailnet üyesi)"]
    end

    DC01 <-->|"Tailscale mesh VPN\n(WireGuard, public RDP yok)"| CLIENT01
    Host -.->|tailnet üyesi| DC01
    Host -.->|tailnet üyesi| CLIENT01
```

**Neden bu mimari?** Apple Silicon Mac'lerde VirtualBox yalnızca native ARM64 misafir işletim sistemi çalıştırabiliyor (x86-64 emülasyonu yok), Windows Server'ın ise resmi ARM64 ISO'su yok. Bu yüzden sunucu tarafı Azure'a taşındı (x86-64 native), istemci tarafı yerelde ARM64 olarak kaldı. İkisi arasındaki bağlantı, public RDP portu açmak yerine Tailscale (WireGuard tabanlı mesh VPN) üzerinden kuruldu — gerçek kurumsal ortamlarda tercih edilen, daha güvenli bir yaklaşım.

## Kullanılan teknolojiler

| Bileşen | Detay |
|---|---|
| Domain Controller | Windows Server 2022 Datacenter, Azure (D2s_v6) |
| İstemci | Windows 11 ARM64, VirtualBox 7.2+ (yerel) |
| Dizin servisi | Active Directory Domain Services (AD DS), DNS Server |
| Ağ | Tailscale (WireGuard mesh VPN) — public RDP kapalı |
| Domain | `corp.local` |

## Ne yapıldı (özet)

1. Azure'da DC01 VM'i oluşturuldu, NSG ile erişim kısıtlandı
2. VirtualBox'ta CLIENT01 (Windows 11 ARM64) yerel olarak kuruldu
3. Her iki makineye Tailscale kurulup aynı tailnet'e bağlandı (Mac host dahil)
4. DC01'de AD DS rolü kuruldu, `corp.local` ormanı oluşturuldu, DC01 domain controller'a yükseltildi
5. CLIENT01, `corp.local` domain'ine katıldı ve `Test-ComputerSecureChannel -Verbose` ile kriptografik olarak doğrulandı (`True`)
6. Geçici RDP/NSG kuralları kaldırıldı, tüm erişim Tailscale üzerinden sağlandı

## Kurulum kanıtları

**AD DS kurulum öncesi ön koşul kontrolü.** İki uyarı çıktı: biri Windows NT 4.0 uyumlu zayıf şifreleme algoritmalarının varsayılan olarak kapalı olduğunu söylüyor (bu iyi bir şey, dokunmadım), diğeri ağ arayüzünde sabit IP olmadığından şikâyet ediyor. İkincisi Azure'da beklenen bir uyarı — bulutta IP adresi işletim sistemi içinden değil, Azure NIC ayarlarından sabitleniyor.

![AD DS ön koşul kontrolü](screenshots/03-ad-ds-prerequisites-check.png)

**DC01 kurulum sonrası.** AD DS, DNS ve File and Storage Services rolleri ayakta; sunucu artık `corp.local` domain controller'ı.

![Server Manager — AD DS ve DNS kurulu](screenshots/04-ad-ds-dns-kurulu-dashboard.png)

**CLIENT01 domain katılımının doğrulanması.** Arayüzdeki "hoş geldiniz" mesajına güvenmek yerine güven ilişkisini komutla test ettim:

![Test-ComputerSecureChannel True](screenshots/08-securechannel-true-dogrulama.png)

## Karşılaşılan hatalar

Kurulum sürecinde 9 ayrı gerçek hatayla karşılaştım — mimari uyumsuzluktan yanlış rol kurulumuna, yanlış yorumlanan arayüz mesajlarından, süreç boyunca not tutmak için kullandığım yapay zekâ asistanının kendi kendine yaptığı yanlış bir çıkarıma kadar. Hepsi ekran görüntüleriyle birlikte burada:

👉 **[ERRORS.md — Karşılaştığım Hatalar](ERRORS.md)**

En öğretici ikisi (7. ve 8.): ikisi de "sistem bana başarılı olduğunu söyledi ama aslında değildi" temasını paylaşıyor — biri notlarımı tutan yapay zekâ asistanının doğrulamadan yaptığı bir çıkarım, diğeri Windows'un kendisinin yanıltıcı bir arayüz mesajıydı. İkisinde de gerçek durumu ancak canlı sistemde doğrulayarak (`Get-ADDomain`, `Test-ComputerSecureChannel -Verbose`) teyit edebildim.

## İletişim

Eren Çatak — [LinkedIn](https://www.linkedin.com/in/eren-%C3%A7atak-7539b5222)
