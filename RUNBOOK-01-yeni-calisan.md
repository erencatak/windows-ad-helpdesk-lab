# Runbook #1 - Yeni Çalışan Geldiğinde

(ilk hali )

## Ne zaman lazım

Yeni biri işe başladığında AD'de hesabını açmak için.

## Yöntem 1 - tek kişi, hızlıca (GUI)

Tek bir kişi ekleyeceksem script falan uğraştırmaya değmez, direkt:

1. `dsa.msc` aç (Active Directory Users and Computers)
2. Departmanlar OU'suna sağ tık > New > User
3. Ad, soyad, kullanıcı adı gir
4. Next, şifre koy, "user must change password at next logon" işaretli kalsın
5. Finish
6. Sonra o kullanıcıya sağ tık > Add to a group > hangi departmansa onun grubunu yaz (IT, Satis, Muhasebe, IK, Pazarlama)

Bitti. Tek kişi için bu kadar yeterli.

## Yöntem 2 - birden fazla kişi varsa (script)

İK'dan "bu ay şu kadar kişi başlıyor" diye bir liste geldiğinde tek tek GUI'den açmak yerine:

1. Önce bir CSV hazırla, şu formatta:
   ```
   Ad,Soyad,KullaniciAdi,Departman
   ```
2. `C:\AD-Scripts\` klasöründe `bulk-create-users.ps1` scripti var, onu çalıştır:
   ```powershell
   & "C:\AD-Scripts\bulk-create-users.ps1"
   ```
3. Script CSV'yi okuyup hepsini otomatik oluşturuyor, gruba da ekliyor.

Script'in kendisi zaten `C:\AD-Scripts\bulk-create-users.ps1` içinde duruyor, gerekirse aç bak.

## Kontrol

Gerçekten oluşmuş mu diye:
```powershell
Get-ADUser -Filter * -SearchBase "OU=Departmanlar,DC=corp,DC=local" | Select-Object Name, SamAccountName
```

## Not

- Şifreyi ilk girişte değiştirtiyoruz, bunu unutma
- Departman gruplarının isimleri: IT, Satis, Muhasebe, IK, Pazarlama - CSV'deki Departman sütunu bunlarla birebir aynı yazılmalı yoksa Add-ADGroupMember hata verir

---
*eksikler: hata durumunda ne yapılacağı yazılmadı, hangi yetkiye sahip olman gerektiği yazılmadı, ekran görüntüsü yok. sonra eklerim.*
