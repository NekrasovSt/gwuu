# gwuu (go web ultimate utils)
This is a sel of ultimate utils when using GO language to develop web applications.

Contains following tools:
* gorm - set of functions to extend gorm features:
    - build connection string (Postgres, Mssql, Mysql)
    - create database (Postgres, Mssql, Mysql)
    - drop database  (Postgres, Mssql, Mysql)
    - get next identifier (sometimes GORM is unable to create entities with auto-generated identifiers therefore i have to gen it manually)
    - get portion of data

* testingutils - a set of utils that makes unit testing easily

## Gorm
As were mentioned above we could make connection to any database (MySql, MsSql or Postgres) with creation specified database before, drop database. This functionality
is especially useful in unit tests. I.e. (see unit tests for more details)

```go
func TestPostgresOpenDbWithCreate(t *testing.T) {
    // Create Db when open
	cfg := gorm.Config{}
	connStr := BuildConnectionString(Postgres, "127.0.0.1", 5432, "gwuu_examples", dbUser, dbPassword, "disable")
	testOpenDbWithCreateAndCheck(t, connStr, Postgres, &cfg)
}

func TestMysqlOpenDbWithCreate(t *testing.T) {
	cfg := gorm.Config{}
	connStr := BuildConnectionString(Mysql, "127.0.0.1", 3306, "gwuu_examples", dbUser, dbPassword, "")
	testOpenDbWithCreateAndCheck(t, connStr, Mysql, &cfg)
}

func TestMssqlOpenDbWithCreate(t *testing.T) {
	cfg := gorm.Config{}
	connStr := BuildConnectionString(Mssql, "localhost", 1433, "GwuuExamples", dbUser, dbPassword, "")
	testOpenDbWithCreateAndCheck(t, connStr, Mssql, &cfg)
}

func testOpenDbWithCreateAndCheck(t *testing.T, connStr string, dialect SqlDialect, options *gorm.Config) {
	db := OpenDb2(dialect, connStr, true, options)
	assert.NotNil(t, db)
	// Close
	CloseDb(db)
	// Drop
	DropDb(dialect, connStr)
	// Check
	checkResult := CheckDb(dialect, connStr)
	assert.Equal(t, false, checkResult)
}
```

## Testingutils

Contains following features:
* a set of CheckType functions that allow to compare **ARRAYS** of primitive types i.e. CheckStrings, CheckFloats64, CheckComplexes & others for comparison 
  all following primitive types:
    - []int
    - []int32
    - []uint32
    - []int64
    - []uint64
    - []float32
    - []float64
    - []complex64
    - []complex128

All ***arrays could be compared with ORDER or WITHOUT it*** (for more details please see checkers_test). And one more thing that should be noted - all Check functions
contains assertErr parameter which means if it was set to True error will be asserted via `assert.Equal` or `assert.True` function

`floats and complex types could be compared with tollerance` i.e.:
```go
func TestCheckFloat64SuccessfulWithOrder(t *testing.T) {
	arr1 := make([]float64, 3)
	arr1[0] = 10.55
	arr1[1] = 99
	arr1[2] = 55.9

	arr2 := make([]float64, 3)
	arr2[0] = 11.01
	arr2[1] = 99
	arr2[2] = 55.6

	checkResult, err := CheckFloats64(t, arr1, arr2, 0.5, true, true)
	assert.True(t, checkResult)
	assert.Empty(t, err)
}

func TestCheckComplex64SuccessfulWithOrder(t *testing.T) {
	arr1 := make([]complex64, 3)
	arr1[0] = complex(10.55, 9.12)
	arr1[1] = complex(99, 101)
	arr1[2] = complex(55.9, 67.23)

	arr2 := make([]complex64, 3)
	arr2[0] = complex(11.01, 9.49)
	arr2[1] = complex(99, 100.91)
	arr2[2] = complex(55.6, 66.88)

	checkResult, err := CheckComplexes(t, arr1, arr2, 0.5, true, true)
	assert.True(t, checkResult)
	assert.Empty(t, err)
}
```


# Useful materials
* our article on medium about how to work with https://m-ushakov.medium.com/intricacies-of-working-with-gorm-3d336f310
* our Yandex Zen article: https://zen.yandex.ru/media/id/5f9bbb6eb5987b74e7014a6f/osobennosti-raboty-s-gorm-605234513eb679416826d74c
