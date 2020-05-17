# Android
## Build the user interface

### 1. ViewModel
- **ViewModel** : là đối tượng cung cấp data cho những component UI riêng, như là fragment hay activit, và chứa business logic để kiểm soát dư liệu và giao tiếp với model. Nó có thể gọi component khác để load data , forward request của user đến giữ liệu đã chỉnh sửa . Nó không ảnh hưởng khi configuge thay đổi.


**Chúng ta cần định nghĩa những file sau:**
* **user_profile.xml** : The UI layout definition for the screen.


* **UserProfileViewModel:** 
  * Kế thừa ViewModel. 
  * The class that prepares the data for viewing in the UserProfileFragment and reacts to user interactions.
  * **Lưu dữ data :**
    - User ID : dùng để định danh cho user 
    - User object : class data chứa thông tin chi tiết về user
```js
class UserProfileViewModel : ViewModel() {
   val userId : String = TODO()
   val user : User = TODO()
}
```

* **UserProfileFragment** : The UI controller that displays the data.
```js
class UserProfileFragment : Fragment() {
   private val viewModel: UserProfileViewModel by viewModels()

   override fun onCreateView(
       inflater: LayoutInflater, container: ViewGroup?,
       savedInstanceState: Bundle?
   ): View {
       return inflater.inflate(R.layout.main_fragment, container, false)
   }
}
```
-------------------------------------------------------------------------
### 2. SavedState module

Để có được **user** , **ViewModel** cần phải access vào các đối số của **Fragment**. Ta có thể gửi chúng từ Fragment, nhưng tốt hớn nên dùng **SavedState module**, chúng ta có thể làm cho **ViewModel** của chúng ta đọc trực tiếp đối số.

```js
// UserProfileViewModel
class UserProfileViewModel(
    //SavingStateHandle cho phép ViewModel truy cập trạng thái đã lưu và các đối số của Fragment hoặc Activity liên quan.
   savedStateHandle: SavedStateHandle
) : ViewModel() {
   val userId : String = savedStateHandle["uid"] ?:
          throw IllegalArgumentException("missing user id")
   val user : User = TODO()
}
```

```js
//UserProfileFragment
class UserProfileFragment : Fragment() {
   private val viewModel: UserProfileViewModel by viewModels(
        //Thêm đoạn này
        factoryProducer = { SavedStateVMFactory(this) }
        ...
   )

   override fun onCreateView(
       inflater: LayoutInflater, container: ViewGroup?,
       savedInstanceState: Bundle?
   ): View {
       return inflater.inflate(R.layout.main_fragment, container, false)
   }
}
```

>Bây giờ cần thông báo cho Fragment khi có được đối tượng **user**. Đây là nơi mà thành phần kiến ​​trúc **LiveData** xuất hiện.
-------------------------------------------------------------------------
### 3. LiveData

**LiveData** là một observable data holder. Các component của ứng dụng có thể quan sát được sử thay đổi của đối tượng bằng cách sử dụng holder này, mà không cần phải tạo các liên kết lằng nhằng, giúp logic rõ ràng hơn, và dọn các tham chiếu khi không cần nữa.

**Để sử dụng LiveData :**
- Thay đổi field type trong **UserProfileViewModel**
```js
class UserProfileViewModel(
   savedStateHandle: SavedStateHandle
) : ViewModel() {
   val userId : String = savedStateHandle["uid"] ?:
          throw IllegalArgumentException("missing user id")

   //Thay đổi field type trong UserProfileViewModel thành LiveData<User>
   val user : LiveData<User> = TODO()
}
```
> Bây giờ **UserProfileFragment** sẽ được thông báo khi data được **update**.

- Sửa đổi **UserProfileFragment** để quan sát dữ liệu và cập nhật UI:

```js
//UserProfileFragment
class UserProfileFragment : Fragment() {
   private val viewModel: UserProfileViewModel by viewModels(
        //Thêm đoạn này
        factoryProducer = { SavedStateVMFactory(this) }
        ...
   )

   override fun onCreateView(
       inflater: LayoutInflater, container: ViewGroup?,
       savedInstanceState: Bundle?
   ): View {
       return inflater.inflate(R.layout.main_fragment, container, false)
   }

    // Thêm đoạn này để quan sát data và update UI
   override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.user.observe(viewLifecycleOwner) {
            // có thay đổi thì update UI
        }
    }
}
```
-------------------------------------------------------------------------
## Lấy dữ liệu (Retrofit)

Chúng ta đã sử dụng LiveData để kết nối UserProfileViewModel với UserProfileFragment, làm cách nào để có thể tìm nạp dữ cho đối tượng user?

Giả sử rằng backend của ta cung cấp REST API. Chúng ta sử dụng thư viện Retrofit để truy cập vào backend.

Định nghĩa **Webservice** để giao tiếp với **Backend**:
```js
interface Webservice {
   /**
    * @GET declares an HTTP GET request
    * @Path("user") annotation on the userId parameter marks it as a
    * replacement for the {user} placeholder in the @GET path
    */
   @GET("/users/{user}")
   fun getUser(@Path("user") userId: String): Call<User>
}
```
**ViewModel** sẽ ủy thác quá trình tìm nạp dữ liệu cho một module mới, một **repository**.

### 1. Repository

**Repository** modules sẽ kiểm soát việc sử lý và tìm nạp dữ liệu:
- Chúng cung cấp 1 API để phần còn lại của ứng dụng có thể truy xuất dữ liệu một cách dễ dàng.
- Chúng biết lấy dữ liệu từ đâu và API nào sẽ được gọi khi dữ liệu được update.
- Là trung gian giữ các nguồn dữ liệu
- Nó trừu tượng hóa các nguồn dữ liệu từ phần còn lại của ứng dụng.

**UserRepository**
```js
class UserRepository {
   // Sử dụng một instance của WebService để tìm nạp dữ liệu của user
   private val webservice: Webservice = TODO()
   // ...
   fun getUser(userId: String): LiveData<User> {
       // Đây không phải là một thực hiện tối ưu. Chúng tôi sẽ sửa nó sau.
       val data = MutableLiveData<User>()
       webservice.getUser(userId).enqueue(object : Callback<User> {
           override fun onResponse(call: Call<User>, response: Response<User>) {
               data.value = response.body()
           }
           // Error case is left out for brevity.
           override fun onFailure(call: Call<User>, t: Throwable) {
               TODO()
           }
       })
       return data
   }
}
```
-------------------------------------------------------------------------

## Kết nối ViewModel và repository

Bây giờ, sửa đổi **UserProfileViewModel** để sử dụng đối tượng **UserReprepository**:

```js
class UserProfileViewModel @Inject constructor(
   savedStateHandle: SavedStateHandle,
   //Kết nối ViewModel và repository
   userRepository: UserRepository
) : ViewModel() {
   val userId : String = savedStateHandle["uid"] ?:
          throw IllegalArgumentException("missing user id")
   //Kết nối ViewModel và repository
   val user : LiveData<User> = userRepository.getUser(userId)
}
```

-------------------------------------------------------------------------
## Cache data

Việc triển khai UserRepository trừu tượng hóa cuộc gọi đến đối tượng Webservice, nhưng vì nó chỉ dựa vào một nguồn dữ liệu, nên nó không linh hoạt.

Vấn đề chính với việc triển khai UserRepository là sau khi nó tìm nạp dữ liệu từ backend API , nó không lưu trữ dữ liệu đó ở bất cứ đâu. Do đó, nếu người dùng rời khỏi UserProfileFragment, sau đó quay lại với nó, ứng dụng phải tìm nạp lại dữ liệu, ngay cả khi nó không thay đổi.
- Nó lãng phí băng thông.
- Nó buộc người dùng phải đợi truy vấn mới hoàn thành.

Để giải quyết những thiếu sót này, ta thêm một nguồn dữ liệu mới vào UserRepository, nơi lưu trữ các đối tượng User trong bộ nhớ:

**UserRepository**

```js
// Thông báo cho Dagger rằng lớp này chỉ nên được xây dựng một lần.
@Singleton
class UserRepository @Inject constructor(
   private val webservice: Webservice,
   // Simple in-memory cache. Details omitted for brevity.
   private val userCache: UserCache
) {
   fun getUser(userId: String): LiveData<User> {
       val cached : LiveData<User> = userCache.get(userId)
       if (cached != null) {
           return cached
       }
       val data = MutableLiveData<User>()
      
       // Đối tượng LiveData hiện đang trống, nhưng bạn có thể thêm nó vào cache ở 
       // đây vì nó sẽ chọn dữ liệu chính xác sau khi truy vấn hoàn tất.
       userCache.put(userId, data)
       // Việc thực hiện này vẫn còn tối ưu nhưng tốt hơn trước.
       webservice.getUser(userId).enqueue(object : Callback<User> {
           override fun onResponse(call: Call<User>, response: Response<User>) {
               data.value = response.body()
           }

           // Error case is left out for brevity.
           override fun onFailure(call: Call<User>, t: Throwable) {
               TODO()
           }
       })
       return data
   }
}
```
-------------------------------------------------------------------------
## Persist data

Sử dụng triển khai hiện tại, nếu người dùng xoay thiết bị hoặc rời khỏi và ngay lập tức quay lại ứng dụng, giao diện người dùng hiện tại sẽ hiển thị ngay lập tức vì **repository** sẽ lấy dữ liệu từ **Cache**.

Tuy nhiên, điều gì xảy ra nếu người dùng rời khỏi ứng dụng và quay lại hàng giờ sau đó, sau khi HĐH Android đã giết quá trình? Ta cần tìm nạp lại dữ liệu từ mạng. Quá trình tìm nạp lại này không chỉ làm trải nghiệm người dùng xấu; Nó cũng lãng phí vì nó tiêu thụ dữ liệu di động có giá trị.

Bạn có thể khắc phục sự cố này bằng cách caching các yêu cầu web, nhưng điều đó tạo ra một vấn đề mới: Điều gì xảy ra nếu cùng một dữ liệu người dùng xuất hiện từ một loại yêu cầu khác, chẳng hạn như tìm nạp danh sách bạn bè? Ứng dụng này sẽ hiển thị dữ liệu không nhất quán, gây nhầm lẫn ở mức tốt nhất. Ví dụ: ứng dụng của chúng tôi có thể hiển thị hai phiên bản khác nhau của cùng một dữ liệu của người dùng nếu người dùng thực hiện yêu cầu danh sách bạn bè và yêu cầu người dùng đơn vào các thời điểm khác nhau. Ứng dụng của chúng tôi sẽ cần tìm ra cách hợp nhất dữ liệu không nhất quán này.

Cách thích hợp để xử lý tình huống này là sử dụng một persistent model. Đây là nơi  **Room** xuất hiện.

### Room
**Room** là một thư viện ánh xạ đối tượng cung cấp tính bền vững dữ liệu cục bộ với mã soạn sẵn tối thiểu. Tại thời gian biên dịch, nó xác nhận hợp lệ từng truy vấn đối với lược đồ dữ liệu của bạn, do đó các truy vấn SQL bị hỏng dẫn đến lỗi thời gian biên dịch thay vì lỗi thời gian chạy. **Room** tóm tắt một số chi tiết triển khai cơ bản để làm việc với các bảng và truy vấn SQL thô. Nó cũng cho phép bạn quan sát các thay đổi đối với dữ liệu của cơ sở dữ liệu, bao gồm các bộ sưu tập và tham gia truy vấn, hiển thị các thay đổi đó bằng các đối tượng LiveData. Nó thậm chí còn xác định rõ ràng các ràng buộc thực thi nhằm giải quyết các vấn đề luồng thông thường, chẳng hạn như truy cập bộ nhớ trên luồng chính.

Để sử dụng **Room**, cần xác định **local schema** :
- Đầu tiên, thêm chú thích **@Entity** vào lớp **User data model** 
- Chú thích **@PrimaryKey** vào trường **id** của lớp. 
- Các chú thích này đánh dấu **User** là một table trong DB và id là khóa chính của table

**User**
```js
@Entity
data class User(
   @PrimaryKey private val id: String,
   private val name: String,
   private val lastName: String
)
```

Sau đó, tạo một lớp database bằng cách triển khai RoomDatabase :
**UserDatabase**
```js
@Database(entities = [User::class], version = 1)
abstract class UserDatabase : RoomDatabase()
```


Bây giờ cần một cách để insert dữ liệu user vào database. Chúng ta cần tạo một đối tượng **DAO**.
**UserDao**
```js
@Dao
interface UserDao {
   @Insert(onConflict = REPLACE)
   fun save(user: User)

   //Lưu ý rằng phương thức load trả về một đối tượng kiểu LiveData <User>
   @Query("SELECT * FROM user WHERE id = :userId")
   fun load(userId: String): LiveData<User>
}
```

Với lớp **UserDao** ,tham chiếu DAO từ lớp **database** được sửa:

```js
//UserDatabase

@Database(entities = [User::class], version = 1)
abstract class UserDatabase : RoomDatabase() {
   // thêm đoạn code này vào
   abstract fun userDao(): UserDao
}
```

Bây giờ sửa đổi **UserRepository** để kết hợp nguồn dữ liệu **Room**:

```js
// Thông báo cho Dagger rằng lớp này chỉ nên được xây dựng một lần.
@Singleton
class UserRepository @Inject constructor(
   private val webservice: Webservice,
   // Không sài cache nữa mà sai Room
   private val executor: Executor,
   private val userDao: UserDao
) {
   fun getUser(userId: String): LiveData<User> {
       refreshUser(userId)
       // Returns a LiveData object directly from the database.
       return userDao.load(userId)
   }

   private fun refreshUser(userId: String) {
       // Runs in a background thread.
       executor.execute {
           // Check if user data was fetched recently.
           val userExists = userDao.hasUser(FRESH_TIMEOUT)
           if (!userExists) {
               // Refreshes the data.
               val response = webservice.getUser(userId).execute()

               // Check for errors here.

               // Updates the database. The LiveData object automatically
               // refreshes, so we don't need to do anything else here.
               userDao.save(response.body()!!)
           }
       }
   }

   companion object {
       val FRESH_TIMEOUT = TimeUnit.DAYS.toMillis(1)
   }
}
```

**Lưu ý :** rằng mặc dù chúng ta đã thay đổi nơi dữ liệu đến từ **UserRepository**, nhưng không cần thay đổi **UserProfileViewModel** hoặc **UserProfileFragment**.
