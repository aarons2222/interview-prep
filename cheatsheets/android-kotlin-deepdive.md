# Android / Kotlin Deep Dive Cheat Sheet

---

## 🔄 Activity & Fragment Lifecycle

### Activity Lifecycle
Understanding this is crucial — interviewers love asking about it.

```
         ┌─────────────┐
         │  onCreate   │  ← Activity created (setup UI, initialize)
         └──────┬──────┘
                ▼
         ┌─────────────┐
         │  onStart    │  ← Visible to user
         └──────┬──────┘
                ▼
         ┌─────────────┐
         │  onResume   │  ← Interactive (foreground)
         └──────┬──────┘
                │
        [USER INTERACTING]
                │
                ▼
         ┌─────────────┐
         │  onPause    │  ← Partially obscured (dialog, multi-window)
         └──────┬──────┘
                ▼
         ┌─────────────┐
         │  onStop     │  ← No longer visible
         └──────┬──────┘
                ▼
         ┌─────────────┐
         │  onDestroy  │  ← Being destroyed (cleanup)
         └─────────────┘
```

**Key points:**
- **onCreate** → Setup views, bind data, one-time initialization
- **onStart** → Register listeners, start animations
- **onResume** → Resume paused operations (camera, sensors)
- **onPause** → Pause operations, save critical data
- **onStop** → Release resources, unregister listeners
- **onDestroy** → Final cleanup, release all resources

**Configuration changes (rotation):**
Activity destroyed and recreated → `onDestroy` → `onCreate`
- Use **ViewModel** to survive this
- Or handle manually: `android:configChanges="orientation|screenSize"`

### Fragment Lifecycle
Same as Activity, plus:
- `onAttach` → Attached to activity
- `onCreateView` → Inflate layout
- `onViewCreated` → View ready, setup UI
- `onDestroyView` → View destroyed (but fragment may live)
- `onDetach` → Detached from activity

---

## 🧠 ViewModel & LiveData

### Why ViewModel?
- Survives configuration changes (rotation)
- Separates UI logic from UI controller
- Scoped to Activity/Fragment lifecycle

```kotlin
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user  // Expose immutable
    
    private val _isLoading = MutableLiveData(false)
    val isLoading: LiveData<Boolean> = _isLoading
    
    fun loadUser(id: String) {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                val result = userRepository.getUser(id)
                _user.value = result
            } catch (e: Exception) {
                // Handle error
            } finally {
                _isLoading.value = false
            }
        }
    }
}
```

### Observing LiveData
```kotlin
class UserActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel.user.observe(this) { user ->
            // Update UI
            binding.nameText.text = user.name
        }
        
        viewModel.isLoading.observe(this) { loading ->
            binding.progressBar.isVisible = loading
        }
        
        viewModel.loadUser("123")
    }
}
```

### StateFlow (Modern Alternative)
```kotlin
class UserViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun loadUser() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            val user = userRepository.getUser()
            _uiState.update { it.copy(user = user, isLoading = false) }
        }
    }
}

data class UiState(
    val user: User? = null,
    val isLoading: Boolean = false,
    val error: String? = null
)

// In Activity/Fragment:
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { state ->
            // Update UI
        }
    }
}
```

**LiveData vs StateFlow:**
| LiveData | StateFlow |
|----------|-----------|
| Android-specific | Kotlin standard |
| Lifecycle-aware by default | Need `repeatOnLifecycle` |
| No initial value required | Requires initial value |
| Simpler for basic cases | More powerful, composable |

---

## 🔀 Coroutines Deep Dive

### Basics
```kotlin
// Launch a coroutine (fire and forget)
lifecycleScope.launch {
    val data = fetchData()  // Suspends, doesn't block
    updateUI(data)
}

// Async (returns result)
lifecycleScope.launch {
    val deferred = async { fetchData() }
    val result = deferred.await()
}
```

### Dispatchers
```kotlin
// Main thread (UI)
withContext(Dispatchers.Main) {
    textView.text = "Hello"
}

// IO operations (network, disk)
withContext(Dispatchers.IO) {
    val response = api.fetchUsers()
    database.saveUsers(response)
}

// CPU-heavy work
withContext(Dispatchers.Default) {
    val sorted = hugeList.sortedBy { it.name }
}
```

### Structured Concurrency
```kotlin
// viewModelScope automatically cancels when ViewModel cleared
class MyViewModel : ViewModel() {
    fun doWork() {
        viewModelScope.launch {
            // Cancelled when ViewModel dies
        }
    }
}

// lifecycleScope cancels when lifecycle ends
class MyFragment : Fragment() {
    fun doWork() {
        lifecycleScope.launch {
            // Cancelled when fragment destroyed
        }
    }
}
```

### Error Handling
```kotlin
viewModelScope.launch {
    try {
        val user = userRepository.getUser()
        _user.value = user
    } catch (e: IOException) {
        _error.value = "Network error"
    } catch (e: Exception) {
        _error.value = "Unknown error"
    }
}

// Or with supervisorScope (one failure doesn't cancel siblings)
supervisorScope {
    val users = async { fetchUsers() }
    val posts = async { fetchPosts() }  // Continues even if users fails
}
```

### Parallel Execution
```kotlin
// Sequential (slow)
val user = fetchUser()      // 2 seconds
val posts = fetchPosts()    // 2 seconds
// Total: 4 seconds

// Parallel (fast)
coroutineScope {
    val user = async { fetchUser() }
    val posts = async { fetchPosts() }
    
    val result = Pair(user.await(), posts.await())
    // Total: 2 seconds
}
```

---

## 🎨 Jetpack Compose

### Basic Composables
```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}

@Composable
fun UserCard(user: User) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = user.name,
                style = MaterialTheme.typography.headlineSmall
            )
            Text(
                text = user.email,
                style = MaterialTheme.typography.bodyMedium
            )
        }
    }
}
```

### State Management
```kotlin
// remember - survives recomposition
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}

// rememberSaveable - survives configuration changes
@Composable
fun SearchBar() {
    var query by rememberSaveable { mutableStateOf("") }
    
    TextField(
        value = query,
        onValueChange = { query = it },
        placeholder = { Text("Search...") }
    )
}
```

### State Hoisting
Move state up, pass down values and lambdas:

```kotlin
// Stateless (reusable)
@Composable
fun CounterDisplay(
    count: Int,
    onIncrement: () -> Unit
) {
    Button(onClick = onIncrement) {
        Text("Count: $count")
    }
}

// Stateful (owns state)
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    
    CounterDisplay(
        count = count,
        onIncrement = { count++ }
    )
}
```

### With ViewModel
```kotlin
@Composable
fun UserScreen(
    viewModel: UserViewModel = viewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    
    when {
        uiState.isLoading -> LoadingSpinner()
        uiState.error != null -> ErrorMessage(uiState.error!!)
        else -> UserContent(uiState.user)
    }
}
```

### Lists
```kotlin
@Composable
fun UserList(users: List<User>) {
    LazyColumn {
        items(
            items = users,
            key = { it.id }  // Important for performance
        ) { user ->
            UserCard(user)
        }
    }
}
```

### Side Effects
```kotlin
// Run once when composable enters composition
LaunchedEffect(Unit) {
    viewModel.loadData()
}

// Run when key changes
LaunchedEffect(userId) {
    viewModel.loadUser(userId)
}

// Cleanup when leaving composition
DisposableEffect(Unit) {
    val listener = createListener()
    onDispose {
        listener.remove()
    }
}
```

---

## 💉 Dependency Injection (Hilt)

### Setup
```kotlin
// Application class
@HiltAndroidApp
class MyApp : Application()

// Module
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_db"
        ).build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(): ApiService {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}
```

### Injection
```kotlin
// In Activity
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    @Inject
    lateinit var analytics: Analytics
}

// In ViewModel
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository
) : ViewModel() {
    // userRepository automatically injected
}

// In Repository
class UserRepository @Inject constructor(
    private val api: ApiService,
    private val db: AppDatabase
) {
    // Dependencies injected
}
```

---

## 💾 Room Database

### Setup
```kotlin
// Entity
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    val name: String,
    val email: String,
    @ColumnInfo(name = "created_at") val createdAt: Long
)

// DAO
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAll(): Flow<List<UserEntity>>
    
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getById(id: String): UserEntity?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: UserEntity)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(users: List<UserEntity>)
    
    @Delete
    suspend fun delete(user: UserEntity)
    
    @Query("DELETE FROM users")
    suspend fun deleteAll()
}

// Database
@Database(entities = [UserEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

### Usage
```kotlin
class UserRepository @Inject constructor(
    private val userDao: UserDao,
    private val api: ApiService
) {
    // Observe database changes as Flow
    fun observeUsers(): Flow<List<User>> {
        return userDao.getAll().map { entities ->
            entities.map { it.toUser() }
        }
    }
    
    // Fetch from network, cache in DB
    suspend fun refreshUsers() {
        val users = api.getUsers()
        userDao.insertAll(users.map { it.toEntity() })
    }
}
```

---

## 🌐 Networking with Retrofit

### Setup
```kotlin
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<UserDto>
    
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: String): UserDto
    
    @POST("users")
    suspend fun createUser(@Body user: CreateUserRequest): UserDto
    
    @PUT("users/{id}")
    suspend fun updateUser(
        @Path("id") id: String,
        @Body user: UpdateUserRequest
    ): UserDto
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: String)
    
    @GET("search")
    suspend fun search(
        @Query("q") query: String,
        @Query("page") page: Int = 1
    ): SearchResponse
}
```

### With Interceptors
```kotlin
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor { chain ->
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        chain.proceed(request)
    }
    .addInterceptor(HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    })
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .client(okHttpClient)
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

---

## 🏗️ Clean Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                       │
│  (Activities, Fragments, ViewModels, Composables)           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Domain Layer                           │
│  (Use Cases, Repository Interfaces, Domain Models)          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Data Layer                            │
│  (Repository Impl, Data Sources, API, Database, DTOs)       │
└─────────────────────────────────────────────────────────────┘
```

### Example Flow
```kotlin
// Domain - Use Case
class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository
) {
    suspend operator fun invoke(id: String): Result<User> {
        return userRepository.getUser(id)
    }
}

// Domain - Repository Interface
interface UserRepository {
    suspend fun getUser(id: String): Result<User>
    fun observeUsers(): Flow<List<User>>
}

// Data - Repository Implementation
class UserRepositoryImpl @Inject constructor(
    private val api: ApiService,
    private val dao: UserDao
) : UserRepository {
    
    override suspend fun getUser(id: String): Result<User> {
        return try {
            val user = api.getUser(id)
            dao.insert(user.toEntity())
            Result.success(user.toDomain())
        } catch (e: Exception) {
            // Try local cache
            val cached = dao.getById(id)
            if (cached != null) {
                Result.success(cached.toDomain())
            } else {
                Result.failure(e)
            }
        }
    }
}

// Presentation - ViewModel
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun loadUser(id: String) {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            getUserUseCase(id)
                .onSuccess { _uiState.value = UiState.Success(it) }
                .onFailure { _uiState.value = UiState.Error(it.message) }
        }
    }
}
```

---

## 🧪 Testing

### Unit Tests
```kotlin
@Test
fun `loadUser updates state correctly`() = runTest {
    // Given
    val fakeRepo = FakeUserRepository()
    fakeRepo.setUser(User("1", "Aaron"))
    val viewModel = UserViewModel(GetUserUseCase(fakeRepo))
    
    // When
    viewModel.loadUser("1")
    
    // Then
    val state = viewModel.uiState.value
    assertTrue(state is UiState.Success)
    assertEquals("Aaron", (state as UiState.Success).user.name)
}

@Test
fun `loadUser handles error`() = runTest {
    val fakeRepo = FakeUserRepository()
    fakeRepo.setShouldFail(true)
    val viewModel = UserViewModel(GetUserUseCase(fakeRepo))
    
    viewModel.loadUser("1")
    
    assertTrue(viewModel.uiState.value is UiState.Error)
}
```

### Compose UI Tests
```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun counterIncrements() {
    composeTestRule.setContent {
        CounterScreen()
    }
    
    composeTestRule.onNodeWithText("Count: 0").assertExists()
    composeTestRule.onNodeWithText("Count: 0").performClick()
    composeTestRule.onNodeWithText("Count: 1").assertExists()
}
```

---

## 🔒 ProGuard / R8

```proguard
# Keep data classes for Gson/Retrofit
-keep class com.yourapp.data.model.** { *; }

# Keep Retrofit interfaces
-keep,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}

# Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
```

---

*You've built eLOQ and Unify on Android — you've done all of this. This is just a refresher before the interview. 💪*
