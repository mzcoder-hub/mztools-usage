Berikut adalah langkah-langkah dan potongan kode untuk membuat aplikasi sederhana menggunakan Golang, Gin Gonic, GORM, dan JWT untuk fitur yang Anda sebutkan:

### 1. **Setup Project**
Buat folder untuk proyek Anda, misalnya `hris-app`. Di dalam folder ini, buat struktur seperti berikut:

```
hris-app/
├── main.go
├── config/
│   └── database.go
├── controllers/
│   ├── authController.go
│   ├── employeeController.go
│   └── attendanceController.go
├── models/
│   ├── employee.go
│   ├── user.go
│   └── attendance.go
├── middlewares/
│   └── jwtMiddleware.go
└── routes/
    └── routes.go
```

### 2. **Setup `main.go`**

```go
package main

import (
	"github.com/gin-gonic/gin"
	"hris-app/config"
	"hris-app/routes"
)

func main() {
	r := gin.Default()

	// Setup database connection
	config.SetupDatabase()

	// Setup routes
	routes.SetupRoutes(r)

	r.Run(":8080") // Run on port 8080
}
```

### 3. **Setup Database Connection (`config/database.go`)**

```go
package config

import (
	"fmt"
	"log"
	"os"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"hris-app/models"
)

var DB *gorm.DB

func SetupDatabase() {
	dsn := "user:password@tcp(127.0.0.1:3306)/hris?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatalf("Failed to connect to database: %v", err)
	}

	// Migrate the schema
	err = db.AutoMigrate(&models.Employee{}, &models.User{}, &models.Attendance{})
	if err != nil {
		log.Fatalf("Failed to migrate database: %v", err)
	}

	DB = db
	fmt.Println("Database connection established.")
}
```

### 4. **Models (`models/employee.go`, `models/user.go`, `models/attendance.go`)**

#### `models/employee.go`
```go
package models

import (
	"gorm.io/gorm"
)

type Employee struct {
	gorm.Model
	FirstName   string `json:"first_name"`
	LastName    string `json:"last_name"`
	Photo       string `json:"photo"`
	Contact     string `json:"contact"`
	Education   string `json:"education"`
	Certification string `json:"certification"`
	Experience  string `json:"experience"`
	Position    string `json:"position"`
	Department  string `json:"department"`
	UserID      uint   `json:"user_id"`
}
```

#### `models/user.go`
```go
package models

import (
	"gorm.io/gorm"
)

type User struct {
	gorm.Model
	Username  string    `json:"username" gorm:"unique"`
	Password  string    `json:"password"`
	Employee  Employee  `json:"employee"`
}
```

#### `models/attendance.go`
```go
package models

import (
	"gorm.io/gorm"
)

type Attendance struct {
	gorm.Model
	EmployeeID uint   `json:"employee_id"`
	CheckIn    string `json:"check_in"`
	CheckOut   string `json:"check_out"`
	LeaveType  string `json:"leave_type"`
	LeaveStatus string `json:"leave_status"`
}
```

### 5. **Controllers**

#### `controllers/authController.go`
```go
package controllers

import (
	"net/http"
	"github.com/gin-gonic/gin"
	"hris-app/config"
	"hris-app/models"
	"hris-app/utils"
	"golang.org/x/crypto/bcrypt"
)

func Register(c *gin.Context) {
	var user models.User
	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	hashedPassword, err := bcrypt.GenerateFromPassword([]byte(user.Password), bcrypt.DefaultCost)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	user.Password = string(hashedPassword)

	if err := config.DB.Create(&user).Error; err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"message": "User registered successfully"})
}

func Login(c *gin.Context) {
	var input struct {
		Username string `json:"username"`
		Password string `json:"password"`
	}

	if err := c.ShouldBindJSON(&input); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	var user models.User
	if err := config.DB.Where("username = ?", input.Username).First(&user).Error; err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
		return
	}

	if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(input.Password)); err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
		return
	}

	token, err := utils.GenerateJWT(user.ID)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"token": token})
}
```

#### `controllers/employeeController.go`
```go
package controllers

import (
	"net/http"
	"github.com/gin-gonic/gin"
	"hris-app/config"
	"hris-app/models"
)

func CreateEmployee(c *gin.Context) {
	var employee models.Employee
	if err := c.ShouldBindJSON(&employee); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	if err := config.DB.Create(&employee).Error; err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"employee": employee})
}

func GetEmployees(c *gin.Context) {
	var employees []models.Employee
	if err := config.DB.Find(&employees).Error; err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"employees": employees})
}
```

#### `controllers/attendanceController.go`
```go
package controllers

import (
	"net/http"
	"github.com/gin-gonic/gin"
	"hris-app/config"
	"hris-app/models"
)

func RecordAttendance(c *gin.Context) {
	var attendance models.Attendance
	if err := c.ShouldBindJSON(&attendance); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	if err := config.DB.Create(&attendance).Error; err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"attendance": attendance})
}

func GetAttendances(c *gin.Context) {
	var attendances []models.Attendance
	if err := config.DB.Find(&attendances).Error; err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"attendances": attendances})
}
```

### 6. **JWT Middleware (`middlewares/jwtMiddleware.go`)**

```go
package middlewares

import (
	"net/http"
	"github.com/gin-gonic/gin"
	"hris-app/utils"
)

func JWTAuthMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		err := utils.ValidateJWT(c)
		if err != nil {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "Unauthorized"})
			c.Abort()
			return
		}
		c.Next()
	}
}
```

### 7. **JWT Utility (`utils/jwt.go`)**

```go
package utils

import (
	"fmt"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v4"
)

var jwtKey = []byte("secret_key")

type Claims struct {
	UserID uint `json:"user_id"`
	jwt.StandardClaims
}

func GenerateJWT(userID uint) (string, error) {
	expirationTime := time.Now().Add(24 * time.Hour)
	claims := &Claims{
		UserID: userID,
		StandardClaims: jwt.StandardClaims{
			ExpiresAt: expirationTime.Unix(),
		},
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString(jwtKey)
}

func ValidateJWT(c *gin.Context) error {
	tokenStr := c.Request.Header.Get("Authorization")
	claims := &Claims{}

	token, err := jwt.ParseWithClaims(tokenStr, claims, func(token *jwt.Token) (interface{}, error) {
		return jwtKey, nil
	})

	if err != nil || !token.Valid {
		return fmt.Errorf("invalid token")
	}

	c.Set("user_id", claims.UserID)
	return nil
}
```

### 8. **Setup Routes (`routes/routes.go`)**

```go
package routes



import (
	"github.com/gin-gonic/gin"
	"hris-app/controllers"
	"hris-app/middlewares"
)

func SetupRoutes(r *gin.Engine) {
	r.POST("/register", controllers.Register)
	r.POST("/login", controllers.Login)

	auth := r.Group("/")
	auth.Use(middlewares.JWTAuthMiddleware())
	{
		auth.POST("/employees", controllers.CreateEmployee)
		auth.GET("/employees", controllers.GetEmployees)

		auth.POST("/attendance", controllers.RecordAttendance)
		auth.GET("/attendance", controllers.GetAttendances)
	}
}
```

### 9. **Run the Application**
Setelah semua file diatur, Anda dapat menjalankan aplikasi dengan perintah berikut di terminal:

```bash
go mod init hris-app
go mod tidy
go run main.go
```

Aplikasi ini akan berjalan di `http://localhost:8080`. Anda dapat menguji endpoint seperti `/register`, `/login`, `/employees`, dan `/attendance` menggunakan tools seperti Postman.

### Catatan:
- Jangan lupa untuk mengganti `user:password` pada string koneksi database sesuai dengan konfigurasi MySQL lokal Anda.
- `jwtKey` sebaiknya diubah ke kunci yang lebih aman untuk aplikasi produksi.
- Penggunaan `bcrypt` untuk hashing password memastikan keamanan yang lebih baik.

Ini adalah template dasar yang bisa Anda kembangkan lebih lanjut sesuai kebutuhan rumah sakit.


Untuk menggunakan PostgreSQL sebagai database, Anda perlu melakukan beberapa penyesuaian pada proyek yang telah saya berikan. Berikut adalah langkah-langkah untuk menyesuaikan koneksi database dan skema untuk PostgreSQL.

### 1. **Setup Database Connection (`config/database.go`)**

Ubah koneksi database di `config/database.go` untuk menggunakan PostgreSQL:

```go
package config

import (
	"fmt"
	"log"
	"os"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"hris-app/models"
)

var DB *gorm.DB

func SetupDatabase() {
	// Format dsn PostgreSQL
	dsn := "host=localhost user=your_username password=your_password dbname=hris port=5432 sslmode=disable TimeZone=Asia/Jakarta"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatalf("Failed to connect to database: %v", err)
	}

	// Migrate the schema
	err = db.AutoMigrate(&models.Employee{}, &models.User{}, &models.Attendance{})
	if err != nil {
		log.Fatalf("Failed to migrate database: %v", err)
	}

	DB = db
	fmt.Println("Database connection established.")
}
```

### 2. **Install PostgreSQL Driver**

Pastikan Anda telah menginstal driver PostgreSQL untuk GORM. Jika belum, tambahkan dependensi berikut dalam proyek Anda dengan menjalankan perintah ini di terminal:

```bash
go get -u gorm.io/driver/postgres
```

### 3. **Menyesuaikan `main.go`**

`main.go` akan tetap sama seperti sebelumnya, tetapi pastikan bahwa Anda telah mengkonfigurasi `SetupDatabase` untuk PostgreSQL.

### 4. **Skema Models**

Model-model (`models/employee.go`, `models/user.go`, `models/attendance.go`) tidak memerlukan perubahan khusus untuk PostgreSQL, karena GORM akan secara otomatis menyesuaikan tipe data untuk PostgreSQL berdasarkan struktur model yang Anda tentukan.

### 5. **Database Setup di PostgreSQL**

Pastikan Anda telah membuat database PostgreSQL yang akan digunakan:

1. Masuk ke PostgreSQL:
   ```bash
   psql -U your_username
   ```

2. Buat database baru:
   ```sql
   CREATE DATABASE hris;
   ```

3. Berikan hak akses yang sesuai kepada pengguna:
   ```sql
   GRANT ALL PRIVILEGES ON DATABASE hris TO your_username;
   ```

### 6. **Run the Application**

Setelah semua diatur, jalankan aplikasi dengan:

```bash
go run main.go
```

Aplikasi sekarang akan terhubung ke database PostgreSQL yang Anda buat dan menggunakan PostgreSQL untuk menyimpan data.

### 7. **Testing**

Anda dapat menggunakan alat seperti Postman untuk menguji endpoint seperti `/register`, `/login`, `/employees`, dan `/attendance`.

### Catatan Tambahan

- **`sslmode=disable`** digunakan untuk pengembangan lokal. Dalam produksi, Anda mungkin ingin mengaktifkan SSL untuk keamanan.
- Sesuaikan `TimeZone=Asia/Jakarta` sesuai dengan zona waktu Anda.
- Jangan lupa untuk mengganti `your_username` dan `your_password` dengan kredensial PostgreSQL Anda yang sebenarnya.

Dengan mengikuti langkah-langkah di atas, Anda dapat menjalankan aplikasi HRIS menggunakan PostgreSQL sebagai database back-end.
