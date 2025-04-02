# EFIstatistikProjesi 📊

Bu proje, .NET Framework üzerinde C# ve Windows Forms kullanılarak geliştirilmiş, Entity Framework (Database First yaklaşımı) ile çeşitli istatistiksel sorguların nasıl yapılabileceğini gösteren bir masaüstü uygulamasıdır.

## 🚀 Genel Bakış

Uygulama, Kategori (Category), Ürün (Product), Müşteri (Customer) ve Sipariş (Order) tablolarını içeren bir veritabanı üzerinde Entity Framework LINQ metotları ve `Database.SqlQuery` kullanarak çeşitli istatistiksel verileri hesaplar ve bir form üzerinde görüntüler. Bu proje, Entity Framework'ün sorgulama yeteneklerini ve yaygın istatistiksel işlemleri (Count, Sum, Average, Where, OrderBy, Join vb.) anlamak için bir demo niteliğindedir.

## ✨ Gösterilen İstatistiksel Sorgular ve Özellikler

*   **Temel Sayımlar:**
    *   Toplam Kategori Sayısı (`Count()`)
    *   Toplam Ürün Sayısı (`Count()`)
    *   Toplam Müşteri Sayısı (`Count()`)
    *   Toplam Sipariş Sayısı (`Count()`)
*   **Toplam ve Ortalamalar:**
    *   Toplam Ürün Stok Sayısı (`Sum()`)
    *   Ortalama Ürün Fiyatı (`Average()`)
*   **Koşullu İstatistikler:**
    *   Belirli Bir Kategorideki (Örn: Meyve) Toplam Stok (`Where()`, `Sum()`)
    *   Belirli Bir Ürünün (Örn: Gazoz) Toplam İşlem Hacmi (Fiyat * Stok) (`Where()`, `Select()`, `FirstOrDefault()`)
    *   Stok Sayısı Belirli Bir Değerden Az Olan Ürün Sayısı (`Where()`, `Count()`)
    *   Belirli Bir Kategorideki (Örn: Sebze) Aktif Ürünlerin Toplam Stoğu (`Where()` içinde `Where()`, `Select()`, `FirstOrDefault()`, `Sum()`)
    *   Aktif Ürün Sayısı (`Where()`, `Count()`)
    *   Belirli Bir Ürünün (Örn: Kola) Toplam Stok Değeri (Fiyat * Stok)
*   **Sıralama ve Seçme:**
    *   Son Eklenen Ürünün Adı (`OrderByDescending()`, `Select()`, `FirstOrDefault()`)
    *   Son Eklenen Ürünün Kategori Adı (İki sorgu ile: Önce Ürün, sonra Kategori)
    *   Son Sipariş Veren Müşterinin Adı (İki sorgu ile: Önce Sipariş, sonra Müşteri)
*   **Benzersiz Değerler:**
    *   Farklı Müşteri Ülkelerinin Sayısı (`Select()`, `Distinct()`, `Count()`)
*   **SQL Query vs. Entity Framework:**
    *   Belirli Bir Ülkedeki (Örn: Türkiye) Sipariş Sayısı (Hem `Database.SqlQuery` hem de EF LINQ metodu ile)
    *   Belirli Bir Kategorideki (Örn: Meyve) Ürünlerin Toplam Sipariş Tutarı (Hem `Database.SqlQuery` hem de EF LINQ `Join` ile)

## 🛠️ Kullanılan Teknolojiler

*   **Programlama Dili:** C#
*   **Framework:** .NET Framework 4.7.2
*   **Arayüz:** Windows Forms (WinForms)
*   **Veri Erişimi:** Entity Framework 6 (Database First)
*   **Veritabanı:** Microsoft SQL Server

## 💾 Veritabanı Kurulumu

Uygulamanın çalışması için `EFIstatistikProjesi` adında bir SQL Server veritabanına ve ilgili tablolara ihtiyaç vardır.

1.  **Veritabanı ve Tabloları Oluşturma:** Aşağıdaki SQL scriptlerini bir SQL Server üzerinde çalıştırarak `EFIstatistikProjesi` veritabanını ve gerekli tabloları oluşturun:

    ```sql
    -- Veritabanını oluştur (eğer yoksa)
    IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = 'EFIstatistikProjesi')
    BEGIN
        CREATE DATABASE EFIstatistikProjesi;
    END
    GO

    USE EFIstatistikProjesi;
    GO

    -- TblCategory Tablosu
    IF NOT EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[dbo].[TblCategory]') AND type in (N'U'))
    BEGIN
        CREATE TABLE [dbo].[TblCategory](
            [CategoryId] [int] IDENTITY(1,1) NOT NULL,
            [CategoryName] [nvarchar](50) NULL,
        CONSTRAINT [PK_TblCategory] PRIMARY KEY CLUSTERED ([CategoryId] ASC)
        );
    END
    GO

    -- TblProduct Tablosu
    IF NOT EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[dbo].[TblProduct]') AND type in (N'U'))
    BEGIN
        CREATE TABLE [dbo].[TblProduct](
            [ProductId] [int] IDENTITY(1,1) NOT NULL,
            [ProductName] [nvarchar](50) NULL,
            [ProductStock] [int] NULL,
            [ProductPrice] [decimal](18, 2) NULL,
            [ProductStatus] [bit] NULL,
            [CategoryId] [int] NULL,
        CONSTRAINT [PK_TblProduct] PRIMARY KEY CLUSTERED ([ProductId] ASC)
        );

        ALTER TABLE [dbo].[TblProduct] WITH CHECK ADD CONSTRAINT [FK_TblProduct_TblCategory] FOREIGN KEY([CategoryId])
        REFERENCES [dbo].[TblCategory] ([CategoryId]);

        ALTER TABLE [dbo].[TblProduct] CHECK CONSTRAINT [FK_TblProduct_TblCategory];
    END
    GO

    -- TblCustomer Tablosu
    IF NOT EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[dbo].[TblCustomer]') AND type in (N'U'))
    BEGIN
        CREATE TABLE [dbo].[TblCustomer](
            [CustomerId] [int] IDENTITY(1,1) NOT NULL,
            [CustomerName] [nvarchar](50) NULL,
            [CustomerCountry] [nvarchar](50) NULL,
            [CustomerCity] [nvarchar](50) NULL,
        CONSTRAINT [PK_TblCustomer] PRIMARY KEY CLUSTERED ([CustomerId] ASC)
        );
    END
    GO

    -- TblOrder Tablosu
    IF NOT EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[dbo].[TblOrder]') AND type in (N'U'))
    BEGIN
        CREATE TABLE [dbo].[TblOrder](
            [OrderId] [int] IDENTITY(1,1) NOT NULL,
            [CustomerId] [int] NULL,
            [ProductId] [int] NULL,
            [Count] [int] NULL,
            [UnitPrice] [decimal](18, 2) NULL,
            [TotalPrice] [decimal](18, 2) NULL,
        CONSTRAINT [PK_TblOrder] PRIMARY KEY CLUSTERED ([OrderId] ASC)
        );

        ALTER TABLE [dbo].[TblOrder] WITH CHECK ADD CONSTRAINT [FK_TblOrder_TblCustomer] FOREIGN KEY([CustomerId])
        REFERENCES [dbo].[TblCustomer] ([CustomerId]);

        ALTER TABLE [dbo].[TblOrder] CHECK CONSTRAINT [FK_TblOrder_TblCustomer];

        ALTER TABLE [dbo].[TblOrder] WITH CHECK ADD CONSTRAINT [FK_TblOrder_TblProduct] FOREIGN KEY([ProductId])
        REFERENCES [dbo].[TblProduct] ([ProductId]);

        ALTER TABLE [dbo].[TblOrder] CHECK CONSTRAINT [FK_TblOrder_TblProduct];
    END
    GO

    -- Örnek Veri Ekleme (İsteğe bağlı)
    -- INSERT INTO TblCategory (CategoryName) VALUES ('Meyve'), ('Sebze'), ('İçecek');
    -- INSERT INTO TblProduct (ProductName, ProductStock, ProductPrice, ProductStatus, CategoryId) VALUES ('Elma', 150, 5.50, 1, 1), ('Domates', 200, 4.00, 1, 2), ('Kola', 50, 8.00, 1, 3), ('Gazoz', 80, 7.50, 0, 3), ('Muz', 90, 12.00, 1, 1);
    -- INSERT INTO TblCustomer (CustomerName, CustomerCountry, CustomerCity) VALUES ('Ali Veli', 'Türkiye', 'İstanbul'), ('Ayşe Yılmaz', 'Türkiye', 'Ankara'), ('John Doe', 'USA', 'New York');
    -- INSERT INTO TblOrder (CustomerId, ProductId, Count, UnitPrice, TotalPrice) VALUES (1, 1, 5, 5.50, 27.50), (2, 3, 10, 8.00, 80.00), (1, 5, 3, 12.00, 36.00), (3, 2, 20, 4.00, 80.00);
    ```

2.  **Bağlantı Dizesi (Connection String):** Projenin `App.config` dosyasındaki `connectionStrings` bölümünü kontrol edin. `EFIstatistikProjesiEntities` adlı bağlantı dizesindeki `data source` kısmını kendi SQL Server sunucu adınızla değiştirmeniz gerekebilir. Mevcut ayar: `data source=UMUT\SQLEXPRESS`.

    ```xml
    <connectionStrings>
      <add name="EFIstatistikProjesiEntities" connectionString="metadata=res://*/Model1.csdl|res://*/Model1.ssdl|res://*/Model1.msl;provider=System.Data.SqlClient;provider connection string="data source=YOUR_SERVER_NAME;initial catalog=EFIstatistikProjesi;integrated security=True;trustservercertificate=True;MultipleActiveResultSets=True;App=EntityFramework"" providerName="System.Data.EntityClient" />
    </connectionStrings>
    ```
    `YOUR_SERVER_NAME` kısmını kendi sunucu bilgilerinizle güncelleyin (örneğin, `.` , `(localdb)\mssqllocaldb`, `YOUR_PC_NAME\SQLEXPRESS` vb.).

## 🏃 Nasıl Çalıştırılır?

1.  Projeyi bilgisayarınıza klonlayın:
    ```bash
    git clone https://github.com/kullanici-adiniz/EFIstatistikProjesi.git
    ```
    *(kullanici-adiniz kısmını kendi GitHub kullanıcı adınızla değiştirin)*
2.  Yukarıdaki "Veritabanı Kurulumu" adımlarını takip ederek veritabanını hazırlayın. (Örnek verileri eklemek isteğe bağlıdır ama istatistikleri görmek için faydalıdır).
3.  Gerekirse `App.config` dosyasındaki bağlantı dizesini güncelleyin.
4.  `EFIstatistikProjesi.sln` dosyasını Visual Studio (2019 veya üzeri önerilir) ile açın.
5.  NuGet Paket Yöneticisi'nin Entity Framework paketini geri yüklediğinden emin olun (genellikle otomatik yapılır). Gerekirse `Update-Package -reinstall EntityFramework` komutunu Paket Yöneticisi Konsolu'nda çalıştırın.
6.  Projeyi derleyin (Build -> Build Solution).
7.  Uygulamayı başlatın (Debug -> Start Debugging veya F5). `Form1` açılacak ve istatistikler otomatik olarak hesaplanıp görüntülenecektir.

## 🔄 Modeli Güncelleme (Database First)

Eğer veritabanı şemasında (tablo ekleme/çıkarma, sütun değiştirme vb.) değişiklik yaparsanız, Entity Framework modelini güncellemeniz gerekir:

1.  Visual Studio'da Solution Explorer'da `Model1.edmx` dosyasına çift tıklayarak EDMX Tasarımcısını açın.
2.  Tasarımcı yüzeyinde boş bir alana sağ tıklayın.
3.  `Update Model from Database...` seçeneğini seçin.
4.  Açılan sihirbazda `Refresh` sekmesinden güncellenecek tabloları seçin veya `Add` sekmesinden yeni tabloları ekleyin. `Finish` butonuna tıklayın.
5.  Değişiklikleri kaydedin ve projeyi yeniden derleyin.

---

Bu README, projenin amacını, yeteneklerini ve nasıl kullanılacağını anlamanıza yardımcı olacaktır.
