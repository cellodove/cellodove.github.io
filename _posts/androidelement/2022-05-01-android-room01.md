---
published: true
title : "[Android] Room을 사용하여 로컬 데이터베이스에 데이터 저장1"
excerpt: "Room 라이브러리 설명"

categories :
  - Elements
  
toc: true
toc_sticky: true
last_modified_at: 2022-05-01
---

![room_image1.jpg](/assets/images/room_image1.jpg?raw=true)

## Room이란?

**Android** **Jetpack**의 구성요소로서 안드로이드 기기 내부에 데이터를 저장하기 위해 사용하는 라이브러리이다.

Room은 SQLite를 내부적으로 사용하고 있지만, DB를 구조적으로 분리하여 데이터 접근의 편의성을 높여주고 유지보수에 편리하다.

### Room의 장점

SQlite는 쿼리의 컴파일 시간 검증을 할 수 없다. 그러나 Room에는 컴파일 타임에 SQL 유효성 검사가 있다.

Room은 상용구 코드 없이 데이터베이스 데이터를 Java 또는 Kotlin 객체에 매핑할 수 있다.

Room은 데이터 관찰을 위해 LiveData 및 RxJava와 함께 작동하도록 구축되었다.

## 기본 구성요소

Room에는 다음 3가지 주요 구성요소가 있다.

- [데이터베이스 클래스](https://developer.android.com/reference/kotlin/androidx/room/Database?hl=ko)(DB): 데이터베이스를 보유하고 앱의 영구 데이터와의 기본 연결을 위한 기본 액세스 포인트 역할을 한다.
- [데이터 항목](https://developer.android.com/training/data-storage/room/defining-data?hl=ko)(Entities): 앱 데이터베이스의 테이블을 나타낸다.
- [데이터 액세스 객체(DAO)](https://developer.android.com/training/data-storage/room/accessing-data?hl=ko): 앱이 데이터베이스의 데이터를 쿼리, 업데이트, 삽입, 삭제하는 데 사용할 수 있는 메서드를 제공한다.

![room_image2.jpg](/assets/images/room_image2.jpg?raw=true)

### Entities

DB 내의 Table, 즉 DB에 저장할 데이터 형식으로 class의 변수들이 컬럼(column)이 되어 table이 된다.

**Annotaion**  

- @Entity

Entity요소임을 선언한다. 기본적으로 클래스 이름을 테이블이름으로 사용하나 다른이름으로 설정하고싶으면 @Entity에서 이름을 설정할 수 있다.

- @PrimaryKey

테이블의 고유행을 식별하는 기본 키를 정의해야한다.

- @ColumnInfo

기본적으로 필드 이름을 선언된 변수 이름으로 사용하나. 변수명과 컬럼명을 다르게 하고싶으면 해당 주석에 이름을 설정하면된다.

```kotlin
@Entity(tableName = "users")
data class User (
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

### DAO

데이터베이스에 접근하여 수행할 작업을 메소드 형태로 정의 (SQL 쿼리 지정 가능)

**Annotaion**  

- @Dao

DAO요소임을 선언한다. 어느 경우든 항상 주석을 달아야한다.

- @insert

데이터베이스 테이블에 데이터를 삽입하는 메서드를 정의할 수 있다.

```kotlin
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insertUsers(vararg users: User)

    @Insert
    fun insertBothUsers(user1: User, user2: User)

    @Insert
    fun insertUsersAndFriends(user: User, friends: List<User>)
}
```

- @update

데이터베이스 테이블에서 특정 행을 업데이트하는 메서드를 정의할 수 있다. Return 값으로 업데이트된 행 수를 받을 수 있다.

```kotlin
@Dao
interface UserDao {
    @Update
    fun updateUsers(vararg users: User)
}
```

- @delete

데이터베이스 테이블에서 특정 행을 삭제하는 메서드를 정의할 수 있다. Return 값으로 삭제된 행 수를 받을 수 있다.

```kotlin
@Dao
interface UserDao {
    @Delete
    fun deleteUsers(vararg users: User)
}
```

- @query

SQL 문을 작성하여 DB를 조회할 수 있다.

Compile time 에 리턴되는 object의 field와 sql 결과로 나오는 column의 이름이 맞는지 확인하여 일부가 match되면 warning, match 되는게 없다면 error를 보낸다.

```kotlin
@Query("SELECT * FROM user")
fun loadAllUsers(): Array<User>
```

### Room DB

데이터베이스의 전체적인 소유자 역할, DB 생성 및 버전 관리.

Room DB에서 DAO를 가져와서 객체를 통해 데이터를 CRUD한다.

**Annotaion**  

- @Database

Database임을 선언한다.

**Properties**  

```kotlin
@Database(
  version = 2,
  entities = [User::class],
  autoMigrations = [
    AutoMigration (from = 1, to = 2)
  ]
)
abstract class AppDatabase : RoomDatabase() {}
```

- autoMigrations

두 데이터베이스 버전 간의 자동 이전을 선언하려면 `autoMigrations`속성에 추가한다.

- entities

데이터베이스에 포함된 엔터티 목록이다.

- exportSchema

Room의 Schema 구조를 폴더로 Export 할 수 있다.

- version

Scheme가 바뀔 때 이 version도 바뀌어야 한다.

## 참조

> [https://developer.android.com/codelabs/android-room-with-a-view-kotlin?hl=ko#0](https://developer.android.com/codelabs/android-room-with-a-view-kotlin?hl=ko#0)
>
> [https://developer.android.com/training/data-storage/room?hl=ko](https://developer.android.com/training/data-storage/room?hl=ko)
>
> [https://velog.io/@ryalya/Android-DB-Room이란](https://velog.io/@ryalya/Android-DB-Room%EC%9D%B4%EB%9E%80)
>