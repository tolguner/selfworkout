# SelfWorkout MSSQL Database Setup

Bu dosya MSSQL Server'da yapılan tüm veritabanı işlemlerini içerir.

## ✅ Kurulum Kontrol Listesi

- [ ] MSSQL Server kuruldu ve çalışıyor
- [ ] selfworkout veritabanı oluşturuldu
- [ ] Tüm tablolar oluşturuldu (17 tablo)
- [ ] Roller eklendi (admin, sporcu)
- [ ] Admin kullanıcısı oluşturuldu
- [ ] Kas grupları eklendi (8 adet)
- [ ] Ekipmanlar eklendi (8 adet)
- [ ] Veritabanı bağlantısı test edildi

---

## 1. Veritabanı Oluşturma

```sql
CREATE DATABASE selfworkout;
USE selfworkout;
```

## 2. Tabloları Oluşturma

### Roller Tablosu
```sql
CREATE TABLE Roles (
    id INT PRIMARY KEY IDENTITY(1,1),
    role_name NVARCHAR(20) NOT NULL UNIQUE
);
```

### Kullanıcılar Tablosu
```sql
CREATE TABLE Users (
    id INT PRIMARY KEY IDENTITY(1,1),
    role_id INT NOT NULL,
    username NVARCHAR(50) NOT NULL UNIQUE,
    email NVARCHAR(100) NOT NULL UNIQUE,
    password NVARCHAR(255) NOT NULL,
    name NVARCHAR(100),
    surname NVARCHAR(100),
    birthdate DATE,
    created_at DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (role_id) REFERENCES Roles(id)
);
```

### Kas Grupları Tablosu
```sql
CREATE TABLE MuscleGroups (
    id INT PRIMARY KEY IDENTITY(1,1),
    name NVARCHAR(100) NOT NULL,
    description NVARCHAR(255)
);
```

### Ekipmanlar Tablosu
```sql
CREATE TABLE Equipments (
    id INT PRIMARY KEY IDENTITY(1,1),
    name NVARCHAR(100) NOT NULL,
    description NVARCHAR(255)
);
```

### Egzersizler Tablosu
```sql
CREATE TABLE Exercises (
    id INT PRIMARY KEY IDENTITY(1,1),
    name NVARCHAR(100) NOT NULL,
    description NVARCHAR(255),
    difficulty_level NVARCHAR(20) CHECK (difficulty_level IN ('kolay', 'orta', 'zor')),
    instructions NVARCHAR(MAX),
    created_by INT NOT NULL,
    created_at DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (created_by) REFERENCES Users(id)
);
```

### İlişki Tabloları
```sql
-- Egzersiz-Kas ilişkisi
CREATE TABLE ExerciseMuscles (
    id INT PRIMARY KEY IDENTITY(1,1),
    exercise_id INT,
    muscle_id INT,
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id),
    FOREIGN KEY (muscle_id) REFERENCES MuscleGroups(id)
);

-- Egzersiz-Ekipman ilişkisi
CREATE TABLE ExerciseEquipments (
    id INT PRIMARY KEY IDENTITY(1,1),
    exercise_id INT,
    equipment_id INT,
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id),
    FOREIGN KEY (equipment_id) REFERENCES Equipments(id)
);

-- Egzersiz Etiketleri
CREATE TABLE ExerciseTags (
    id INT PRIMARY KEY IDENTITY(1,1),
    exercise_id INT,
    tag NVARCHAR(100),
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id)
);
```

### Rutin ve Antrenman Tabloları
```sql
-- Egzersiz Rutinleri
CREATE TABLE ExerciseRoutines (
    id INT PRIMARY KEY IDENTITY(1,1),
    user_id INT NOT NULL,
    name NVARCHAR(100),
    description NVARCHAR(255),
    created_at DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (user_id) REFERENCES Users(id)
);

-- Rutin İçindeki Egzersizler
CREATE TABLE RoutineExercises (
    id INT PRIMARY KEY IDENTITY(1,1),
    routine_id INT NOT NULL,
    exercise_id INT NOT NULL,
    exercise_order INT DEFAULT 1,
    set_count INT DEFAULT 3,
    reps_per_set NVARCHAR(50),
    weight_per_set NVARCHAR(50),
    FOREIGN KEY (routine_id) REFERENCES ExerciseRoutines(id),
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id)
);

-- Günlük Antrenmanlar
CREATE TABLE DailyWorkouts (
    id INT PRIMARY KEY IDENTITY(1,1),
    user_id INT NOT NULL,
    workout_date DATE NOT NULL,
    notes NVARCHAR(255),
    status NVARCHAR(20) DEFAULT 'planned' CHECK (status IN ('planned', 'active', 'completed', 'cancelled')),
    FOREIGN KEY (user_id) REFERENCES Users(id)
);

-- Antrenman Egzersizleri
CREATE TABLE WorkoutExercises (
    id INT PRIMARY KEY IDENTITY(1,1),
    daily_workout_id INT NOT NULL,
    exercise_id INT NOT NULL,
    exercise_order INT DEFAULT 1,
    set_count INT,
    reps_per_set NVARCHAR(50),
    weight_per_set NVARCHAR(50),
    FOREIGN KEY (daily_workout_id) REFERENCES DailyWorkouts(id),
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id)
);
```

### İstatistik ve Takip Tabloları
```sql
-- Vücut İstatistikleri
CREATE TABLE BodyStats (
    id INT PRIMARY KEY IDENTITY(1,1),
    user_id INT NOT NULL,
    record_date DATE DEFAULT CAST(GETDATE() AS DATE),
    height DECIMAL(5,2),
    weight DECIMAL(5,2),
    body_fat DECIMAL(5,2),
    muscle_mass DECIMAL(5,2),
    notes NVARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES Users(id)
);

-- Kullanıcı Antrenmanları
CREATE TABLE UserWorkouts (
    id INT PRIMARY KEY IDENTITY(1,1),
    user_id INT NOT NULL,
    routine_id INT NULL,
    exercise_id INT NULL,
    workout_date DATE DEFAULT CAST(GETDATE() AS DATE),
    workout_type NVARCHAR(20) CHECK (workout_type IN ('daily', 'routine')),
    notes NVARCHAR(255),
    status NVARCHAR(20) DEFAULT 'planned' CHECK (status IN ('planned', 'active', 'completed', 'cancelled')),
    duration_minutes INT,
    started_at DATETIME2,
    completed_at DATETIME2,
    FOREIGN KEY (user_id) REFERENCES Users(id),
    FOREIGN KEY (routine_id) REFERENCES ExerciseRoutines(id),
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id)
);

-- Kullanıcı Antrenman Egzersizleri
CREATE TABLE UserWorkoutExercises (
    id INT PRIMARY KEY IDENTITY(1,1),
    user_workout_id INT NOT NULL,
    exercise_id INT NOT NULL,
    set_number INT,
    reps INT,
    weight DECIMAL(5,2),
    notes NVARCHAR(255),
    FOREIGN KEY (user_workout_id) REFERENCES UserWorkouts(id),
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id)
);
```

### Log ve Favori Tabloları
```sql
-- Sistem Logları
CREATE TABLE Logs (
    id INT PRIMARY KEY IDENTITY(1,1),
    user_id INT,
    action NVARCHAR(255),
    action_time DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (user_id) REFERENCES Users(id)
);

-- Favori Egzersizler
CREATE TABLE FavoriteExercises (
    id INT PRIMARY KEY IDENTITY(1,1),
    user_id INT NOT NULL,
    exercise_id INT NOT NULL,
    created_at DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (user_id) REFERENCES Users(id),
    FOREIGN KEY (exercise_id) REFERENCES Exercises(id)
);
```

## 3. Rolleri Ekleme

```sql
-- Rolleri ekle
INSERT INTO Roles (role_name) VALUES ('admin');
INSERT INTO Roles (role_name) VALUES ('sporcu');

-- Kontrol et
SELECT * FROM Roles;
```

**Beklenen Sonuç:**
```
id | role_name
---|----------
1  | admin
2  | sporcu
```

## 4. Admin Kullanıcısı Oluşturma

```sql
-- Admin kullanıcısını ekle
INSERT INTO Users (role_id, username, email, password, name, surname, created_at)
VALUES (1, 'admin', 'admin@selfworkout.com', 'admin123', 'System', 'Administrator', GETDATE());

-- Kontrol et
SELECT 
    u.id,
    u.username,
    u.email,
    u.name,
    u.surname,
    r.role_name,
    u.created_at
FROM Users u
INNER JOIN Roles r ON u.role_id = r.id
WHERE u.username = 'admin';
```

**🔑 Admin Giriş Bilgileri:**
- **Kullanıcı Adı:** admin
- **Şifre:** admin123
- **Email:** admin@selfworkout.com
- **Rol:** admin (role_id = 1)

## 5. Kas Grupları Ekleme

```sql
-- Kas gruplarını ekle
INSERT INTO MuscleGroups (name, description) VALUES 
('Göğüs', 'Pektoralis kasları'),
('Sırt', 'Latissimus dorsi ve trapez kasları'),
('Omuz', 'Deltoid kasları'),
('Biceps', 'Kol ön kasları'),
('Triceps', 'Kol arka kasları'),
('Bacak', 'Quadriceps ve hamstring kasları'),
('Karın', 'Abdominal kasları'),
('Kalça', 'Glutes kasları');

-- Kontrol et
SELECT * FROM MuscleGroups ORDER BY id;
```

## 6. Ekipmanları Ekleme

```sql
-- Ekipmanları ekle
INSERT INTO Equipments (name, description) VALUES 
('Halter', 'Serbest ağırlık'),
('Dumbbell', 'El halterleri'),
('Barbell', 'Olimpik bar'),
('Kablo Makinesi', 'Kablo sistemi'),
('Smith Machine', 'Guided barbell'),
('Treadmill', 'Koşu bandı'),
('Bisiklet', 'Stationary bike'),
('Vücut Ağırlığı', 'Ekipman gerektirmez');

-- Kontrol et
SELECT * FROM Equipments ORDER BY id;
```

## 7. Veritabanı Durumu Kontrol

### Tüm Tabloları Listele
```sql
SELECT TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_TYPE = 'BASE TABLE' 
ORDER BY TABLE_NAME;
```

### Kullanıcı Sayıları
```sql
SELECT 
    r.role_name,
    COUNT(u.id) as user_count
FROM Roles r
LEFT JOIN Users u ON r.id = u.role_id
GROUP BY r.id, r.role_name;
```

### Admin Detayları
```sql
SELECT 
    u.id,
    u.username,
    u.email,
    u.name + ' ' + u.surname as full_name,
    r.role_name,
    u.created_at
FROM Users u
INNER JOIN Roles r ON u.role_id = r.id
WHERE r.role_name = 'admin';
```

## 8. Test Komutları

### Veritabanı Bağlantı Testi
```bash
java -cp "target/classes;target/dependency/*" com.example.selfworkout.test.DatabaseTest
```

### Uygulama Çalıştırma
```bash
./mvnw javafx:run
```

## 📊 Özet Bilgiler

- **Toplam Tablo Sayısı:** 17
- **Admin Kullanıcı Sayısı:** 1
- **Sporcu Kullanıcı Sayısı:** 0 (kayıt ol butonu ile artacak)
- **Kas Grubu Sayısı:** 8
- **Ekipman Sayısı:** 8

## 🔧 Sorun Giderme

### Bağlantı Sorunları
1. MSSQL Server çalışıyor mu?
2. SQL Server Authentication aktif mi?
3. 'sa' kullanıcısının şifresi doğru mu?
4. Windows Firewall MSSQL'e izin veriyor mu?

### Tablo Sorunları
1. Tüm tablolar oluşturuldu mu?
2. Foreign key ilişkileri doğru mu?
3. Identity kolonları çalışıyor mu?

## 📝 Notlar

- **Kayıt Ol Butonu:** Uygulama üzerinden kayıt olan tüm kullanıcılar `role_id = 2` (sporcu) olarak kaydedilecek
- **Admin Rolü:** `role_id = 1`
- **Sporcu Rolü:** `role_id = 2`
- **Güvenlik:** İlk girişten sonra admin şifresini değiştirmeyi unutmayın

---

**Son Güncelleme:** 16 Haziran 2025
**Durum:** Kurulum tamamlandı ✅
