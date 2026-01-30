# Swipe iOS & KMM Architecture Documentation

**Project**: Swipe iOS Application

**Last Updated**: 2025-11-21

**Current Branch**: `staging`

**Architecture Style**: Hybrid SwiftUI + Kotlin Multiplatform Mobile (KMM)

---

## Table of Contents

1. [Project Structure](#project-structure)
2. [Architectural Patterns](#architectural-patterns)
3. [iOS Architecture](#ios-architecture)
4. [KMM Architecture](#kmm-architecture)
5. [Data Flow](#data-flow)
6. [Module Dependencies](#module-dependencies)
7. [Key Components](#key-components)
8. [Third-Party Dependencies](#third-party-dependencies)
9. [Build Configuration](#build-configuration)
10. [Security](#security)

---

## Project Structure

```
swipe-ios/
├── Swipe/                              # iOS Application
│   ├── Swipe.xcodeproj                # Xcode project
│   ├── Swipe.xcworkspace              # Workspace (includes Pods)
│   ├── Podfile                        # CocoaPods dependencies
│   ├── Swipe/                         # iOS source code
│   │   ├── iOSApp.swift              # App entry point
│   │   ├── Presentation/             # UI Layer
│   │   │   ├── Splash/
│   │   │   ├── Account/              # Login, OTP
│   │   │   ├── Navigation/           # Navigation system
│   │   │   ├── Home/                 # Main app screens
│   │   │   │   ├── Dashboard/
│   │   │   │   ├── Bills/            # Invoices, Purchases
│   │   │   │   ├── Products/         # Product management
│   │   │   │   ├── Parties/          # Customers, Vendors
│   │   │   │   └── More/             # Settings
│   │   │   ├── CustomViews/          # 68+ reusable components
│   │   │   ├── Theme/                # Design system
│   │   │   ├── IAP/                  # In-App Purchases
│   │   │   └── Reports/              # Native reports
│   │   ├── Common/                   # Utilities, constants
│   │   ├── Extensions/               # Swift extensions (30+)
│   │   ├── System/                   # System managers
│   │   ├── Network/                  # Network monitoring
│   │   ├── Security/                 # Keychain
│   │   ├── Analytics/                # Firebase, Clarity
│   │   └── Localisation/            # i18n
│   ├── NotificationService/          # Rich notifications extension
│   ├── SwipeWidget/                  # Home screen widget
│   └── SmartechNSEContent/          # Smartech notifications
│
└── shared/                            # KMM Shared Module
    ├── build.gradle.kts              # Kotlin build config
    └── src/
        ├── commonMain/kotlin/in/getswipe/
        │   ├── di/                   # Dependency Injection
        │   │   └── KoinDI.kt        # DI configuration
        │   ├── common/              # Common utilities
        │   │   ├── Constants.kt
        │   │   ├── SharedPreferences.kt
        │   │   └── Utils.kt (expect/actual)
        │   ├── data/                # Data Layer
        │   │   ├── viewmodel/       # 37 ViewModels
        │   │   ├── repository/      # 34 Repository impls
        │   │   ├── remote/          # 33 API services
        │   │   ├── network/         # Network utilities
        │   │   ├── events/          # UI Events
        │   │   └── dummy/           # Test data
        │   └── domain/              # Domain Layer
        │       └── model/
        │           ├── repository/  # Repository interfaces
        │           ├── data/        # Data models
        │           ├── requests/    # 173 API request models
        │           └── responses/   # 165 API response models
        ├── iosMain/kotlin/in/getswipe/
        │   └── common/              # iOS platform implementations (7 files)
        └── androidMain/kotlin/in/getswipe/
            └── common/              # Android platform implementations (8 files)
```

---

## Architectural Patterns

### 1. Clean Architecture

The application follows Clean Architecture principles with clear separation of concerns across three layers:

```
┌─────────────────────────────────────────────────┐
│                  UI Layer                       │
│              (SwiftUI Views)                    │
│  - Pure presentation logic                      │
│  - No business logic                            │
│  - Observes ViewModels                          │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│            Presentation Layer                   │
│          (KMM + Swift ViewModels)               │
│  - Business logic orchestration                 │
│  - State management                             │
│  - UI event handling                            │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│              Domain Layer                       │
│         (Kotlin Interfaces & Models)            │
│  - Repository interfaces                        │
│  - Domain models                                │
│  - Business rules                               │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│               Data Layer                        │
│        (Repository Implementations)             │
│  - API services (Ktor)                          │
│  - Local persistence (Realm)                    │
│  - Data mapping                                 │
└─────────────────────────────────────────────────┘
```

### 2. MVVM (Model-View-ViewModel)

**ViewModel Hierarchy:**

```swift
// KMM ViewModel (Kotlin)
class ProductsViewmodel : ViewModel() {
    val products: MutableLiveData<ResourceState<List<Product>, Exception>>

    fun onUIEvent(events: ProductsUIEvents) {
        when(events) {
            is FetchProducts -> fetchProducts()
            is DeleteProduct -> deleteProduct(events.id)
        }
    }
}

// Swift ViewModel Extension
class ProductsViewmodel: shared.ProductsViewmodel {
    @Published var selectedProduct: Product? = nil
    @Published var shouldNavigateToProductDetailsScreen: Bool = false

    // iOS-specific functionality
    override init() {
        super.init()
        initObservers()
    }

    func initObservers() {
        attachObservers()
    }
}

// SwiftUI View
struct ProductsView: View {
    @EnvironmentViewModel var productsViewmodel: ProductsViewmodel

    var body: some View {
        ResourceStateHandler(
            resourceState: productsViewmodel.products,
            onLoadedData: { products in
                ProductList(products: products)
            }
        )
        .onAppear {
            productsViewmodel.onUIEvent(events: FetchProducts())
        }
    }
}
```

### 3. Repository Pattern

Every data source is abstracted behind a repository interface:

```kotlin
// Domain Layer - Interface
interface ProductsRepository {
    suspend fun getProducts(request: GetProductsRequest): ResourceState<ProductsResponse, Exception>
    suspend fun createProduct(request: CreateProductRequest): ResourceState<CreateProductResponse, Exception>
}

// Data Layer - Implementation
class ProductsRepositoryImpl(
    private val productsApi: ProductsApi
) : ProductsRepository {
    override suspend fun getProducts(request: GetProductsRequest): ResourceState<ProductsResponse, Exception> {
        return try {
            val response = productsApi.getProducts(request)
            NetworkUtils.handleResponse(response)
        } catch (e: Exception) {
            ResourceState.failed(e)
        }
    }
}

// API Service
class ProductsApi(private val httpClient: HttpClient) {
    suspend fun getProducts(request: GetProductsRequest): HttpResponse {
        return httpClient.post {
            url(Constants.Endpoints.GET_PRODUCTS)
            setBody(request)
        }
    }
}
```

### 4. State Management Patterns

#### ResourceState Pattern

All API responses are wrapped in `ResourceState<T, E>`:

```kotlin
sealed class ResourceState<out T, out E> {
    class Loading<T, E> : ResourceState<T, E>()
    data class Success<T, E>(val data: T) : ResourceState<T, E>()
    data class Failed<T, E>(val error: E) : ResourceState<T, E>()
    class Empty<T, E> : ResourceState<T, E>()
}
```

Usage in SwiftUI:

```swift
ResourceStateHandler(
    resourceState: viewModel.companyDetails,
    onLoadingState: { LoadingView() },
    onErrorState: { error in ErrorView(error: error) },
    onLoadedData: { data in ContentView(data: data) }
)
```

#### UI Events Pattern

User interactions are modeled as sealed classes:

```kotlin
sealed class ProductsUIEvents {
    object FetchProducts : ProductsUIEvents()
    data class DeleteProduct(val id: Int) : ProductsUIEvents()
    data class UpdateProduct(val product: Product) : ProductsUIEvents()
}
```

### 5. Dependency Injection (Koin)

**DI Configuration (`KoinDI.kt`):**

```kotlin
fun initKoin() {
    startKoin {
        modules(
            platformDependency,    // HttpClient, Firebase
            apiDependency,         // 33 API services
            repoDependency,        // 34 Repositories
            viewmodelDependency    // 37 ViewModels
        )
    }
}

val apiDependency = module {
    single { ProductsApi(get()) }
    single { CustomerApi(get()) }
    single { DashboardApi(get()) }
    // ... 30 more
}

val repoDependency = module {
    single<ProductsRepository> { ProductsRepositoryImpl(get()) }
    single<CustomerRepository> { CustomerRepositoryImpl(get()) }
    // ... 32 more
}

val viewmodelDependency = module {
    single { ProductsViewmodel() }
    single { CustomersViewModel() }
    // ... 35 more
}
```

**iOS Integration:**

```swift
// Initialize Koin from iOS
func application(...) -> Bool {
    KoinDIKt.doInitKoin()
    return true
}

// Access ViewModels
@StateViewModel var productsVM = DIHelper().productsViewmodel
```

---

## iOS Architecture

### Layer Breakdown

#### 1. Presentation Layer (`/Presentation/`)

**Structure:**
```
Presentation/
├── Splash/                    # App launch & auth check
├── Account/                   # Authentication flow
│   ├── PhoneNumberView
│   ├── OTPView
│   └── WelcomeView
├── Navigation/               # Navigation system
│   ├── Navigator.swift       # Navigation manager
│   ├── NavigationCoordinator.swift
│   └── DeeplinkNavigation.swift
├── Home/                     # Main application
│   ├── Dashboard/           # Landing screen
│   ├── Bills/               # Invoices, Purchases
│   │   ├── CreateBill/
│   │   ├── DocumentDetails/
│   │   └── EWayBills/
│   ├── Products/            # Product catalog
│   ├── Parties/             # Customers, Vendors
│   │   ├── Customers/
│   │   ├── Vendors/
│   │   └── Ledger/
│   └── More/                # Settings
│       ├── CompanyDetails/
│       ├── InvoiceSettings/
│       └── Profile/
├── CustomViews/             # 68+ reusable components
│   ├── SwipeSheet/
│   ├── SwipeCard/
│   ├── ResourceStateHandler/
│   ├── RichTextEditor/
│   └── ...
├── Theme/                   # Design system
│   └── Color.swift
├── IAP/                     # In-App Purchases
│   └── StoreKitService.swift
└── Reports/                 # Native reports
    └── SpreadsheetView/
```

#### 2. Navigation System

**Custom Navigation Manager (`Navigator.swift`):**

```swift
class Navigator: ObservableObject {
    @Published var navigationPath = NavigationPath()
    @Published var stack: Stack<Destination> = []
    @Published var shouldShowSheet = false
    @Published var sheetNavigationPath = NavigationPath()

    enum Destination {
        case Sales
        case Purchases
        case ProductDetailScreen(productId: String)
        case CreateBill(documentType: DocumentTypes, isEdit: Bool)
        case AddCustomerView(isComingFromCreateBillScreen: Bool)
        // 50+ destination types
    }

    func navigate(to destination: Destination) {
        stack.push(destination)
        navigationPath.append(destination)
    }

    func navigateBack() {
        _ = stack.pop()
        if !navigationPath.isEmpty {
            navigationPath.removeLast()
        }
    }

    func navigateBackTo(k: Int) {
        for _ in 0..<k {
            navigateBack()
        }
    }

    func popToRoot() {
        stack.removeAll()
        navigationPath = NavigationPath()
    }

    func navigateSheet(to destination: Destination) {
        shouldShowSheet = true
        sheetNavigationPath.append(destination)
    }
}
```

**Features:**
- Type-safe navigation with enum destinations
- Custom back stack management
- Sheet (modal) navigation support
- Deep linking integration
- Preservation of navigation state

#### 3. Data Persistence

**A. RealmSwift (Primary Database)**

Location: `/Presentation/CustomViews/Realm/`

**Realm Models:**
```swift
class NotesAndTermsDataModalRealm: Object {
    @Persisted(primaryKey: true) var id: String
    @Persisted var notes: String
    @Persisted var terms: String
}

class CompanyBankDetailsRealm: Object {
    @Persisted(primaryKey: true) var id: String
    @Persisted var bankName: String
    @Persisted var accountNumber: String
    @Persisted var ifscCode: String
}

// 20+ Realm models
```

**RealmUtils:**
```swift
extension RealmUtils {
    func saveNotesAndTerms(data: CompanyDetailsResponse) {
        let realm = try! Realm()
        let realmModel = NotesAndTermsDataModalRealm()
        // Convert KMM model to Realm model
        try! realm.write {
            realm.add(realmModel, update: .modified)
        }
    }

    func getNotesAndTerms() -> NotesAndTermsDataModalRealm? {
        let realm = try! Realm()
        return realm.objects(NotesAndTermsDataModalRealm.self).first
    }
}
```

**B. SharedPreferences (UserDefaults)**

Location: `/Common/SharedPrefs.swift`

**Usage:**
```swift
class SharedPrefs {
    static let shared = SharedPrefs()

    var authToken: String? {
        get { UserDefaults.standard.string(forKey: "auth_token") }
        set { UserDefaults.standard.set(newValue, forKey: "auth_token") }
    }

    var companyId: String? {
        get { UserDefaults.standard.string(forKey: "company_id") }
        set { UserDefaults.standard.set(newValue, forKey: "company_id") }
    }

    // Widget data sharing via App Groups
    func saveWidgetData(_ data: WidgetData) {
        let defaults = UserDefaults(suiteName: "group.getswipe.in")
        defaults?.set(data.encoded, forKey: "widget_data")
    }
}
```

**C. Keychain (Secure Storage)**

Location: `/Security/KeychainManager.swift`

**Implementation:**
```swift
protocol KeychainManagerProtocol {
    func saveToken(_ token: String) -> Result<Void, KeychainError>
    func getToken() -> Result<String?, KeychainError>
    func deleteToken() -> Result<Void, KeychainError>
}

final class KeychainManager: KeychainManagerProtocol {
    static let shared = KeychainManager()
    private let service = "com.swipe.mac"
    private let account = "auth_token"

    func saveToken(_ token: String) -> Result<Void, KeychainError> {
        let data = Data(token.utf8)
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]

        let status = SecItemAdd(query as CFDictionary, nil)
        return status == errSecSuccess ? .success(()) : .failure(.saveFailed(status))
    }
}
```

#### 4. Networking

**A. NetworkManager (iOS-specific)**

Location: `/System/NetworkManager.swift`

```swift
class NetworkManager {
    static let shared = NetworkManager()
    private let baseURL = "https://app.getswipe.in/api/"

    func request<T: Decodable>(
        endpoint: String,
        method: HTTPMethod,
        queryParams: [String: String]? = nil,
        requestBody: Encodable? = nil,
        completion: @escaping (Result<T, Error>) -> Void
    ) {
        var components = URLComponents(string: baseURL + endpoint)
        components?.queryItems = queryParams?.map { URLQueryItem(name: $0.key, value: $0.value) }

        guard let url = components?.url else {
            completion(.failure(NetworkError.invalidURL))
            return
        }

        var request = URLRequest(url: url)
        request.httpMethod = method.rawValue
        request.setValue("Bearer \(authToken)", forHTTPHeaderField: "Authorization")
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue(deviceFingerprint, forHTTPHeaderField: "Device_Fingerprint")

        if let body = requestBody {
            request.httpBody = try? JSONEncoder().encode(body)
        }

        URLSession.shared.dataTask(with: request) { data, response, error in
            // Handle response
        }.resume()
    }
}
```

**B. Network Monitoring**

Location: `/Network/NetworkMonitor.swift`

```swift
class NetworkMonitor: ObservableObject {
    @Published var isConnected = true
    private let monitor = NWPathMonitor()

    init() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
            }
        }
        monitor.start(queue: DispatchQueue.global(qos: .background))
    }
}
```

#### 5. In-App Purchases

Location: `/Presentation/IAP/StoreKitService.swift`

```swift
@MainActor
final class StoreKitService: ObservableObject {
    @Published var products: [StoreKit.Product] = []
    @Published var isPurchasing: Bool = false
    @Published var activeSubscription: StoreKit.Product?

    private let productIDs = [
        "com.swipe.premium.monthly",
        "com.swipe.premium.yearly"
    ]

    func loadProducts() async {
        do {
            products = try await Product.products(for: productIDs)
        } catch {
            print("Failed to load products: \(error)")
        }
    }

    func purchase(product: Product, companyId: Int32) async {
        isPurchasing = true
        defer { isPurchasing = false }

        do {
            let result = try await product.purchase()

            switch result {
            case .success(let verification):
                switch verification {
                case .verified(let transaction):
                    // Validate with backend
                    await validatePurchase(transaction, companyId: companyId)
                    await transaction.finish()
                case .unverified:
                    break
                }
            case .userCancelled:
                break
            case .pending:
                break
            @unknown default:
                break
            }
        } catch {
            print("Purchase failed: \(error)")
        }
    }

    private func listenForTransactions() -> Task<Void, Error> {
        return Task.detached {
            for await result in Transaction.updates {
                // Handle transaction updates
            }
        }
    }
}
```

#### 6. Analytics

Location: `/Analytics/AnalyticsManager.swift`

```swift
protocol AnalyticsLogging {
    func log(_ event: AnalyticsEvent)
    func setUserProperty(_ value: String?, for: UserProperty)
    func setUserID(_ id: String?)
}

enum AnalyticsEvent {
    case screenView(name: String, screenClass: String)
    case login(method: LoginMethod, otpToLoginTime: TimeInterval?)
    case onboardingComplete(withGST: Bool, timeTaken: TimeInterval?)
    case firstInvoiceCreated(documentType: String, timeTaken: TimeInterval?)
    case documentCreated(type: String)
    case subscriptionPurchased(plan: String, price: Decimal)
    case featureUsed(feature: String)

    var parameters: [String: Any] {
        switch self {
        case .screenView(let name, let screenClass):
            return ["screen_name": name, "screen_class": screenClass]
        case .login(let method, let time):
            var params: [String: Any] = ["method": method.rawValue]
            if let time = time {
                params["otp_to_login_time"] = time
            }
            return params
        // ... other cases
        }
    }
}

class AnalyticsManager: AnalyticsLogging {
    static let shared = AnalyticsManager()

    private let firebaseAnalytics = FirebaseAnalytics.Analytics.self
    private let clarityAnalytics = Clarity.self

    func log(_ event: AnalyticsEvent) {
        firebaseAnalytics.logEvent(event.name, parameters: event.parameters)
        clarityAnalytics.logCustomEvent(event.name, withProperties: event.parameters)
    }

    func setUserProperty(_ value: String?, for property: UserProperty) {
        firebaseAnalytics.setUserProperty(value, forName: property.rawValue)
    }

    func setUserID(_ id: String?) {
        firebaseAnalytics.setUserID(id)
        if let id = id {
            clarityAnalytics.setCustomUserId(id)
        }
    }
}
```

**Usage:**
```swift
AnalyticsManager.shared.log(.screenView(name: "Dashboard", screenClass: "DashboardView"))
AnalyticsManager.shared.log(.documentCreated(type: "INVOICE"))
```

#### 7. Custom UI Components

68+ reusable components in `/Presentation/CustomViews/`:

**Notable Components:**

1. **SwipeSheet** - Custom bottom sheet
```swift
struct SwipeSheet<Content: View>: View {
    @Binding var isPresented: Bool
    let content: Content

    var body: some View {
        ZStack(alignment: .bottom) {
            if isPresented {
                Color.black.opacity(0.4)
                    .ignoresSafeArea()
                    .onTapGesture { isPresented = false }

                content
                    .transition(.move(edge: .bottom))
            }
        }
    }
}
```

2. **ResourceStateHandler** - Generic state view
```swift
struct ResourceStateHandler<T, Content: View>: View {
    let resourceState: ResourceState<T, Exception>
    let onLoadedData: (T) -> Content

    var body: some View {
        switch resourceState {
        case is ResourceStateLoading:
            LoadingView()
        case let state as ResourceStateSuccess<T, Exception>:
            onLoadedData(state.data)
        case let state as ResourceStateFailed<T, Exception>:
            ErrorView(error: state.error)
        case is ResourceStateEmpty:
            EmptyStateView()
        default:
            EmptyView()
        }
    }
}
```

3. **SwipeEditText** - Custom text field
```swift
struct SwipeEditText: View {
    @Binding var text: String
    let placeholder: String
    let keyboardType: UIKeyboardType
    let isSecure: Bool

    var body: some View {
        if isSecure {
            SecureField(placeholder, text: $text)
        } else {
            TextField(placeholder, text: $text)
                .keyboardType(keyboardType)
        }
        .padding()
        .background(Color.textFieldBackground)
        .cornerRadius(8)
    }
}
```

---

## KMM Architecture

### Layer Breakdown

#### 1. Domain Layer (`domain/model/`)

**Repository Interfaces:**
```kotlin
// domain/model/repository/ProductsRepository.kt
interface ProductsRepository {
    suspend fun getProducts(request: GetProductsRequest): ResourceState<ProductsResponse, Exception>
    suspend fun createProduct(request: CreateProductRequest): ResourceState<CreateProductResponse, Exception>
    suspend fun updateProduct(request: UpdateProductRequest): ResourceState<UpdateProductResponse, Exception>
    suspend fun deleteProduct(request: DeleteProductRequest): ResourceState<DeleteProductResponse, Exception>
}
```

**Data Models:**
```kotlin
// domain/model/data/Product.kt
@Serializable
data class Product(
    @SerialName("id") val id: Int,
    @SerialName("name") val name: String,
    @SerialName("sale_price") val salePrice: Double,
    @SerialName("purchase_price") val purchasePrice: Double?,
    @SerialName("stock_quantity") val stockQuantity: Int?,
    @SerialName("description") val description: String?,
    @SerialName("hsn_code") val hsnCode: String?,
    @SerialName("tax_rate") val taxRate: Double?,
    @SerialName("unit") val unit: String?,
    @SerialName("category") val category: ProductCategory?
)
```

**Request/Response Models:**
- 173 request models in `domain/model/requests/`
- 165 response models in `domain/model/responses/`

#### 2. Data Layer (`data/`)

**Repository Implementations:**
```kotlin
// data/repository/ProductsRepositoryImpl.kt
class ProductsRepositoryImpl(
    private val productsApi: ProductsApi
) : ProductsRepository {

    override suspend fun getProducts(request: GetProductsRequest): ResourceState<ProductsResponse, Exception> {
        return try {
            val response = productsApi.getProducts(request)
            NetworkUtils.handleResponse(response)
        } catch (e: JsonConvertException) {
            FirebaseCrashlytics.recordException(e)
            ResourceState.failed(Exception("Invalid response format"))
        } catch (e: Exception) {
            FirebaseCrashlytics.recordException(e)
            ResourceState.failed(e)
        }
    }

    override suspend fun createProduct(request: CreateProductRequest): ResourceState<CreateProductResponse, Exception> {
        return try {
            val response = productsApi.createProduct(request)
            NetworkUtils.handleResponse(response)
        } catch (e: Exception) {
            ResourceState.failed(e)
        }
    }
}
```

**API Services:**
```kotlin
// data/remote/ProductsApi.kt
class ProductsApi(private val httpClient: HttpClient) {

    suspend fun getProducts(request: GetProductsRequest): HttpResponse {
        return httpClient.post {
            url(Constants.Endpoints.GET_PRODUCTS)
            setBody(request)
        }
    }

    suspend fun createProduct(request: CreateProductRequest): HttpResponse {
        return httpClient.post {
            url(Constants.Endpoints.CREATE_PRODUCT)
            setBody(request)
        }
    }

    suspend fun updateProduct(request: UpdateProductRequest): HttpResponse {
        return httpClient.post {
            url(Constants.Endpoints.UPDATE_PRODUCT)
            setBody(request)
        }
    }

    suspend fun deleteProduct(request: DeleteProductRequest): HttpResponse {
        return httpClient.post {
            url(Constants.Endpoints.DELETE_PRODUCT)
            setBody(request)
        }
    }
}
```

**ViewModels:**
```kotlin
// data/viewmodel/ProductsViewmodel.kt
class ProductsViewmodel : ViewModel() {

    private val productsRepository: ProductsRepository by inject()

    val products = MutableLiveData<ResourceState<ProductsResponse, Exception>>(ResourceState.loading())
    val createProductResponse = MutableLiveData<ResourceState<CreateProductResponse, Exception>>(ResourceState.empty())

    fun onUIEvent(events: ProductsUIEvents) {
        when (events) {
            is ProductsUIEvents.FetchProducts -> fetchProducts(events.request)
            is ProductsUIEvents.CreateProduct -> createProduct(events.request)
            is ProductsUIEvents.UpdateProduct -> updateProduct(events.request)
            is ProductsUIEvents.DeleteProduct -> deleteProduct(events.request)
            is ProductsUIEvents.SearchProducts -> searchProducts(events.query)
        }
    }

    private fun fetchProducts(request: GetProductsRequest) {
        viewModelScope.launch {
            products.postValue(ResourceState.loading())
            val result = productsRepository.getProducts(request)
            products.postValue(result)
        }
    }

    private fun createProduct(request: CreateProductRequest) {
        viewModelScope.launch {
            createProductResponse.postValue(ResourceState.loading())
            val result = productsRepository.createProduct(request)
            createProductResponse.postValue(result)
        }
    }
}
```

**UI Events:**
```kotlin
// data/events/ProductsUIEvents.kt
sealed class ProductsUIEvents {
    data class FetchProducts(val request: GetProductsRequest) : ProductsUIEvents()
    data class CreateProduct(val request: CreateProductRequest) : ProductsUIEvents()
    data class UpdateProduct(val request: UpdateProductRequest) : ProductsUIEvents()
    data class DeleteProduct(val request: DeleteProductRequest) : ProductsUIEvents()
    data class SearchProducts(val query: String) : ProductsUIEvents()
}
```

#### 3. Network Layer

**Ktor HttpClient Configuration:**
```kotlin
// di/KoinDI.kt
val platformDependency = module {
    single {
        httpClient {
            install(ContentNegotiation) {
                json(Json {
                    prettyPrint = true
                    isLenient = true
                    ignoreUnknownKeys = true
                    encodeDefaults = true
                    explicitNulls = false
                    coerceInputValues = true
                    useAlternativeNames = true
                })
            }

            install(Logging) {
                logger = Logger.DEFAULT
                level = LogLevel.ALL
            }

            install(HttpTimeout) {
                requestTimeoutMillis = 60000
                connectTimeoutMillis = 60000
                socketTimeoutMillis = 60000
            }

            defaultRequest {
                val token = SharedPreferences.getAuthToken()
                if (token != null) {
                    header("Authorization", "Bearer $token")
                }
                header("Device_Fingerprint", Utils.getDeviceId())
                header("User-Agent", Utils.getUserAgent())
                header("Request-Timestamp", Utils.getCurrentDateTime())
                header("X-Request-ID", Utils.getPlatformUniqueId())
                header("source", "ios_${Constants.APP_VERSION}")
            }
        }
    }
}
```

**Network Response Handling:**
```kotlin
// data/network/NetworkUtils.kt
object NetworkUtils {
    suspend inline fun <reified T> handleResponse(response: HttpResponse): ResourceState<T, Exception> {
        return when (response.status.value) {
            in 200..299 -> {
                try {
                    val data = response.body<T>()
                    ResourceState.success(data)
                } catch (e: JsonConvertException) {
                    FirebaseCrashlytics.recordException(e)
                    ResourceState.failed(Exception("Failed to parse response"))
                }
            }
            401 -> {
                // Unauthorized - logout user
                SharedPreferences.clearAuthToken()
                ResourceState.failed(Exception("Session expired"))
            }
            429 -> {
                // Too many requests
                ResourceState.failed(Exception("Too many requests, please try again"))
            }
            in 500..511 -> {
                // Server error
                SharedPreferences.setServerDown(true)
                ResourceState.failed(Exception("Server is down"))
            }
            else -> {
                val errorBody = response.bodyAsText()
                val error = parseError(errorBody)
                ResourceState.failed(Exception(error.message))
            }
        }
    }

    private fun parseError(errorBody: String): SwipeError {
        return try {
            Json.decodeFromString<SwipeError>(errorBody)
        } catch (e: Exception) {
            SwipeError(message = "An error occurred")
        }
    }
}
```

#### 4. Platform-Specific Code (expect/actual)

**Example: Utils.kt**

```kotlin
// commonMain/common/Utils.kt
expect object Utils {
    fun getDeviceId(): String
    fun getCurrentDateTime(): String
    fun getUserAgent(): String
    fun getPlatformUniqueId(): String
}

// iosMain/common/Utils.kt
actual object Utils {
    actual fun getDeviceId(): String {
        return UIDevice.currentDevice.identifierForVendor?.UUIDString ?: ""
    }

    actual fun getCurrentDateTime(): String {
        val formatter = NSDateFormatter()
        formatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
        return formatter.stringFromDate(NSDate())
    }

    actual fun getUserAgent(): String {
        val device = UIDevice.currentDevice
        return "Swipe iOS/${Constants.APP_VERSION} (${device.model}; iOS ${device.systemVersion})"
    }

    actual fun getPlatformUniqueId(): String {
        return NSUUID().UUIDString
    }
}

// androidMain/common/Utils.kt
actual object Utils {
    actual fun getDeviceId(): String {
        return Settings.Secure.getString(context.contentResolver, Settings.Secure.ANDROID_ID)
    }

    actual fun getCurrentDateTime(): String {
        return SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault()).format(Date())
    }

    actual fun getUserAgent(): String {
        return "Swipe Android/${Constants.APP_VERSION} (${Build.MODEL}; Android ${Build.VERSION.RELEASE})"
    }

    actual fun getPlatformUniqueId(): String {
        return UUID.randomUUID().toString()
    }
}
```

**Other expect/actual Declarations:**
- `HttpClient.kt` - Platform-specific HTTP engines
- `File.kt` - File handling (UIImage vs ContentResolver)
- `Firebase.kt` - Firebase initialization
- `Log.kt` - Logging implementation
- `DateUtils.kt` - Date parsing
- `TimestampUtils.kt` - Timestamp conversion
- `SharedPreferences.kt` - Platform storage

#### 5. State Management

**MOKO MVVM Integration:**

```kotlin
// ViewModels extend MOKO's ViewModel
abstract class ViewModel : dev.icerock.moko.mvvm.viewmodel.ViewModel()

// LiveData for reactive state
class DashboardViewModel : ViewModel() {
    val companyDetails = MutableLiveData<ResourceState<CompanyResponse, Exception>>(ResourceState.loading())
    val dashboardStats = MutableLiveData<ResourceState<DashboardStatsResponse, Exception>>(ResourceState.empty())
}
```

**iOS Observation:**
```swift
class DashboardVM: shared.DashboardViewModel {
    @Published var isLoading = false

    private var companyObserver: Observer<ResourceState<CompanyResponse, KotlinException>>?

    override init() {
        super.init()
        initObservers()
    }

    func initObservers() {
        companyObserver = Observer { [weak self] state in
            DispatchQueue.main.async {
                self?.isLoading = state is ResourceStateLoading
            }
        }
        self.companyDetails.addObserver(observer: companyObserver!)
    }

    deinit {
        self.companyDetails.removeObserver(observer: companyObserver!)
    }
}
```

---

## Data Flow

### End-to-End Request Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Action                             │
│                   (Button tap in SwiftUI)                       │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      SwiftUI View                               │
│   productsViewmodel.onUIEvent(events: FetchProducts())          │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              Swift ViewModel (iOS Extension)                    │
│    Forwards event to KMM ViewModel, manages iOS-specific state  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                  KMM ViewModel (Kotlin)                         │
│    fun onUIEvent(events: ProductsUIEvents) {                    │
│        when(events) {                                           │
│            is FetchProducts -> fetchProducts()                  │
│        }                                                        │
│    }                                                            │
│    products.postValue(ResourceState.loading())                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                  Repository Interface (Domain)                  │
│    interface ProductsRepository {                               │
│        suspend fun getProducts(...): ResourceState<...>         │
│    }                                                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              Repository Implementation (Data)                   │
│    class ProductsRepositoryImpl(                                │
│        private val productsApi: ProductsApi                     │
│    ) : ProductsRepository {                                     │
│        override suspend fun getProducts(...) {                  │
│            val response = productsApi.getProducts(request)      │
│            return NetworkUtils.handleResponse(response)         │
│        }                                                        │
│    }                                                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                   API Service (Ktor)                            │
│    class ProductsApi(private val httpClient: HttpClient) {      │
│        suspend fun getProducts(request: GetProductsRequest) {   │
│            return httpClient.post {                             │
│                url(Constants.Endpoints.GET_PRODUCTS)            │
│                setBody(request)                                 │
│            }                                                    │
│        }                                                        │
│    }                                                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                  Ktor HttpClient                                │
│    - Adds Authorization header                                  │
│    - Adds Device_Fingerprint                                    │
│    - Adds User-Agent                                            │
│    - Serializes request body to JSON                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  Backend API  │
                    └───────┬───────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                  Response Handling                              │
│    - Deserializes JSON to Kotlin data class                     │
│    - Checks HTTP status code                                    │
│    - Wraps in ResourceState                                     │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              Repository Returns Result                          │
│    ResourceState.success(ProductsResponse)                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│          ViewModel Updates LiveData                             │
│    products.postValue(ResourceState.success(data))              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│        Swift ViewModel Observer Triggered                       │
│    Observer receives new state, updates @Published properties   │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              SwiftUI View Updates                               │
│    ResourceStateHandler detects state change, re-renders UI     │
└─────────────────────────────────────────────────────────────────┘
```

### Offline Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      App Launch                                 │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              Load Cached Data from Realm                        │
│    RealmUtils.getCompanyDetails()                               │
│    RealmUtils.getProducts()                                     │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              Display Cached Data to User                        │
│    (Immediate UI response)                                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│         Check Network Connectivity                              │
│    NetworkMonitor.isConnected                                   │
└───────────────┬───────────────────────┬─────────────────────────┘
                │                       │
    ┌───────────▼──────────┐   ┌────────▼────────────┐
    │   No Connection      │   │   Connected         │
    │   Show cached data   │   │   Fetch fresh data  │
    └──────────────────────┘   └────────┬────────────┘
                                        │
                          ┌─────────────▼──────────────┐
                          │   API Call Success         │
                          │   Save to Realm            │
                          │   Update UI                │
                          └────────────────────────────┘
```

---

## Module Dependencies

### Dependency Graph

```
┌──────────────────────────────────────────────────────────────────┐
│                          iOS App                                 │
│                      (SwiftUI Views)                             │
└────────────────────┬────────────────────┬────────────────────────┘
                     │                    │
        ┌────────────▼──────────┐  ┌──────▼────────────┐
        │ iOS ViewModels        │  │ Custom UI         │
        │ (Swift Extensions)    │  │ Components        │
        └────────────┬──────────┘  └───────────────────┘
                     │
        ┌────────────▼──────────────────────────┐
        │      KMM ViewModels (Kotlin)          │
        │   - 37 ViewModels                     │
        │   - MOKO MVVM                         │
        └────────────┬──────────────────────────┘
                     │
        ┌────────────▼──────────────────────────┐
        │   Repository Interfaces (Domain)      │
        │   - 34 Repository Interfaces          │
        └────────────┬──────────────────────────┘
                     │
        ┌────────────▼──────────────────────────┐
        │  Repository Implementations (Data)    │
        │   - 34 Repository Implementations     │
        └────────────┬──────────────────────────┘
                     │
        ┌────────────▼──────────────────────────┐
        │      API Services (Ktor)              │
        │   - 33 API Service Classes            │
        └────────────┬──────────────────────────┘
                     │
        ┌────────────▼──────────────────────────┐
        │         HttpClient                    │
        │   - Ktor Engine (Darwin/OkHttp)       │
        │   - JSON Serialization                │
        │   - Request/Response Interceptors     │
        └────────────┬──────────────────────────┘
                     │
                     ▼
              Backend REST API
```

### iOS-Specific Dependencies

```
┌─────────────────────────────────────────────────────────┐
│                    iOS Application                      │
└─────────┬───────────────────────────┬───────────────────┘
          │                           │
┌─────────▼──────────┐      ┌─────────▼──────────┐
│ RealmSwift         │      │ KeychainManager    │
│ (Local DB)         │      │ (Secure Storage)   │
└────────────────────┘      └────────────────────┘
          │
┌─────────▼──────────┐
│ NetworkManager     │
│ (iOS Networking)   │
└────────────────────┘
          │
┌─────────▼──────────┐
│ StoreKitService    │
│ (IAP)              │
└────────────────────┘
          │
┌─────────▼──────────┐
│ AnalyticsManager   │
│ (Firebase+Clarity) │
└────────────────────┘
```

### KMM Module Dependencies

```
┌─────────────────────────────────────────────────────────┐
│                  KMM Shared Module                      │
└─────────┬───────────────────────────┬───────────────────┘
          │                           │
┌─────────▼──────────┐      ┌─────────▼──────────┐
│ Koin               │      │ Ktor Client        │
│ (DI)               │      │ (Networking)       │
└────────────────────┘      └────────────────────┘
          │
┌─────────▼──────────┐
│ kotlinx.           │
│ serialization      │
│ (JSON)             │
└────────────────────┘
          │
┌─────────▼──────────┐
│ MOKO MVVM          │
│ (ViewModel+LiveData│
└────────────────────┘
          │
┌─────────▼──────────┐
│ multiplatform-     │
│ settings           │
│ (SharedPreferences)│
└────────────────────┘
```

### Dependency Injection Flow

```
┌─────────────────────────────────────────────────────────┐
│                    KoinDI.kt                            │
│                  initKoin()                             │
└─────────┬───────────────────────────┬───────────────────┘
          │                           │
┌─────────▼──────────┐      ┌─────────▼──────────┐
│ platformDependency │      │ apiDependency      │
│ - HttpClient       │      │ - 33 API Services  │
│ - Firebase         │      │                    │
└────────────────────┘      └────────────────────┘
          │                           │
┌─────────▼──────────┐      ┌─────────▼──────────┐
│ repoDependency     │      │ viewmodelDependency│
│ - 34 Repositories  │      │ - 37 ViewModels    │
└────────────────────┘      └────────────────────┘
          │                           │
          └───────────┬───────────────┘
                      │
          ┌───────────▼────────────┐
          │  Koin Container        │
          │  (Dependency Graph)    │
          └───────────┬────────────┘
                      │
          ┌───────────▼────────────┐
          │  iOS Access via        │
          │  DIHelper()            │
          │  or direct injection   │
          └────────────────────────┘
```

---

## Key Components

### 1. Dashboard Module

**Location**: `/Presentation/Home/Dashboard/`

**Responsibilities**:
- Company selection and switching
- Display dashboard statistics (sales, purchases, expenses)
- Financial year management
- Quick actions (Create Bill, Add Party)
- Deep link handling

**Key Files**:
- `DashboardView.swift` - Main UI
- `DashboardVM.swift` - iOS ViewModel extension
- `shared.DashboardViewModel` - KMM ViewModel

**Features**:
- Real-time stats updates
- Company switcher
- Financial year picker
- Quick access to common actions
- Notification badge handling

### 2. Bills/Documents Module

**Location**: `/Presentation/Home/Bills/`

**Sub-modules**:
- **CreateBill**: Invoice/Purchase creation with line items
- **DocumentDetails**: View/edit existing documents
- **EWayBills**: GST e-way bill generation (India-specific)
- **DocumentList**: Transaction listings with filters

**Key ViewModels**:
- `CreateBillVM` - Document creation logic
- `DocumentsDetailsVM` - Document viewing/editing
- `EWayBillsViewModel` - E-way bill management
- `DocumentVM` - Document list & filtering

**Features**:
- Multi-line item support
- Tax calculation (GST, VAT)
- Discount application
- Payment terms
- Signature capture
- PDF generation
- E-invoicing integration
- E-way bill generation

### 3. Products Module

**Location**: `/Presentation/Home/Products/`

**Responsibilities**:
- Product catalog management
- Inventory tracking
- Batch/serial number handling
- Product variants
- Product grouping
- HSN code management

**Key ViewModels**:
- `ProductsViewmodel` - Product list & CRUD
- `VariantVM` - Product variants management
- `SearchProductVM` - Product search

**Features**:
- Bulk product import
- Product variants (size, color, etc.)
- Inventory timeline tracking
- Low stock alerts
- Barcode scanning
- Product custom columns
- Multi-unit support

### 4. Parties Module

**Location**: `/Presentation/Home/Parties/`

**Sub-modules**:
- **Customers**: Customer management
- **Vendors**: Vendor management
- **Ledger**: Party transaction history
- **PartyPayments**: Payment recording

**Key ViewModels**:
- `CustomersVM` - Customer CRUD
- `VendorsVM` - Vendor CRUD
- `LedgerVM` - Transaction ledger
- `PartyPaymentsVM` - Payment management

**Features**:
- Party CRUD operations
- Billing/shipping addresses
- Credit limit management
- Outstanding balance tracking
- Payment history
- Party merge functionality
- Bulk party import

### 5. Settings/More Module

**Location**: `/Presentation/Home/More/`

**Settings Categories**:
- Company details
- Invoice settings (templates, numbering)
- Document settings (terms, notes)
- Bank details
- Signatures
- General settings
- Product custom columns
- Inventory timeline
- Auto reminders
- User profile

**Key Features**:
- Multi-company management
- Custom invoice templates
- Auto-increment invoice numbers
- Default payment terms
- Company logo upload
- Digital signature management
- Tax settings (GST, VAT)
- Currency settings
- Language preferences

### 6. In-App Purchases Module

**Location**: `/Presentation/IAP/`

**Responsibilities**:
- Subscription management
- Transaction validation
- Receipt handling
- Plan upgrade flows

**Key Components**:
- `StoreKitService.swift` - IAP logic
- `SubscriptionView.swift` - Subscription UI
- `PlanComparisonView.swift` - Plan comparison

**Features**:
- Monthly/yearly subscriptions
- Free trial support
- Promo codes
- Subscription restoration
- Receipt validation
- Backend sync
- Transaction history

### 7. Analytics Module

**Location**: `/Analytics/`

**Tracking Events**:
- Screen views
- User actions (create bill, add party)
- Feature usage
- Conversion events
- Errors and crashes
- Performance metrics

**Integrations**:
- Firebase Analytics
- Firebase Crashlytics
- Microsoft Clarity

**Key Features**:
- Protocol-based abstraction
- Type-safe event logging
- User property tracking
- No PII in events
- A/B testing support (Firebase Remote Config)

---

## Third-Party Dependencies

### iOS Dependencies (CocoaPods)

| Pod | Version | Purpose |
|-----|---------|---------|
| AEOTPTextField | 1.2.6 | OTP input field |
| RichEditorView | 4.0 | Rich text editing (custom fork) |
| Smartech-iOS-SDK | 3.5.5 | Push notifications & analytics |

### iOS Dependencies (SPM - Inferred)

| Package | Purpose |
|---------|---------|
| Firebase Core | Firebase initialization |
| Firebase Crashlytics | Crash reporting |
| Firebase Messaging | Push notifications |
| Firebase Analytics | Analytics tracking |
| Firebase Performance | Performance monitoring |
| Firebase Remote Config | Feature flags |
| RealmSwift | Local database |
| SDWebImage | Image caching |
| SDWebImageSwiftUI | SwiftUI image loading |
| IQKeyboardManagerSwift | Keyboard handling |
| Intercom | Customer support chat |
| CobrowseIO | Co-browsing support |

### KMM Dependencies (Gradle)

| Library | Version | Purpose |
|---------|---------|---------|
| Ktor Client | Latest | Cross-platform HTTP client |
| Koin | Latest | Dependency injection |
| MOKO MVVM | 0.15.0 | ViewModel framework |
| MOKO KSwift | 0.6.1 | Swift API generation |
| kotlinx.serialization | 1.5.1 | JSON serialization |
| multiplatform-settings | 1.0.0 | Shared preferences |
| KMP Native Coroutines | 1.0.0-ALPHA-10 | Coroutines for iOS |
| swipe-calculation-engine | 0.2.0.21-beta | Business logic calculations |

### Framework Versions

```kotlin
// build.gradle.kts
kotlin("multiplatform") version "1.9.10"
android.gradle.plugin version "8.1.2"
```

---

## Build Configuration

### iOS Build Configuration

**Xcode Project**: `Swipe.xcodeproj`
**Workspace**: `Swipe.xcworkspace`

**Targets**:
1. **Swipe** - Main application
2. **NotificationService** - Rich notification handling
3. **SwipeWidget** - Home screen widget
4. **SmartechNSEContent** - Smartech notification extension

**Bundle Identifier**: `com.swipe.mac`

**Configurations**:
- Debug
- Release
- TestFlight (custom)

**Build Phases**:
1. Compile Swift sources
2. Link KMM framework
3. Embed frameworks
4. Copy resources
5. Run CocoaPods script

### KMM Build Configuration

**Build File**: `shared/build.gradle.kts`

**iOS Framework Configuration**:
```kotlin
cocoapods {
    summary = "Shared module for Swipe"
    homepage = "Link to the Shared Module homepage"
    ios.deploymentTarget = "14.0"

    framework {
        baseName = "shared"
        isStatic = true
        export(libs.mokoMvvmCore)
        export(libs.mokoMvvmLiveData)
        export(libs.mokoMvvmLiveDataResources)
        export(libs.mokoMvvmState)
    }
}
```

**iOS Targets**:
```kotlin
val iosX64 = iosX64()
val iosArm64 = iosArm64()
val iosSimulatorArm64 = iosSimulatorArm64()
```

**Gradle Tasks**:
- `./gradlew :shared:linkDebugFrameworkIosArm64` - Build for devices
- `./gradlew :shared:linkDebugFrameworkIosX64` - Build for Intel simulator
- `./gradlew :shared:linkDebugFrameworkIosSimulatorArm64` - Build for M1/M2 simulator

---

## Security

### 1. Authentication & Token Management

**Token Storage**:
- **Previously**: UserDefaults (being migrated)
- **Currently**: Keychain via `KeychainManager`

**KeychainManager Implementation**:
```swift
func saveToken(_ token: String) -> Result<Void, KeychainError> {
    let data = Data(token.utf8)
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecValueData as String: data,
        kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    ]

    // Delete existing item first
    SecItemDelete(query as CFDictionary)

    let status = SecItemAdd(query as CFDictionary, nil)
    return status == errSecSuccess ? .success(()) : .failure(.saveFailed(status))
}
```

**Security Features**:
- `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` - Token only accessible when device unlocked
- Not backed up to iCloud
- Device-specific storage
- Automatic cleanup on app deletion

### 2. Network Security

**HTTPS Only**:
- All API calls use HTTPS
- TLS 1.2+ required
- Certificate pinning (recommended for future)

**Request Headers**:
```kotlin
defaultRequest {
    header("Authorization", "Bearer $token")
    header("Device_Fingerprint", Utils.getDeviceId())
    header("User-Agent", Utils.getUserAgent())
    header("Request-Timestamp", Utils.getCurrentDateTime())
    header("X-Request-ID", Utils.getPlatformUniqueId())
}
```

**Security Measures**:
- Bearer token authentication
- Device fingerprinting
- Request ID tracking
- Timestamp validation
- User-Agent verification

### 3. Data Protection

**Local Data**:
- Realm database encrypted (optional, recommended)
- Keychain for sensitive credentials
- UserDefaults for non-sensitive preferences

**PII Handling**:
- No PII in analytics events
- User consent for data collection
- GDPR compliance measures
- Data export capability
- Data deletion support

### 4. Session Management

**Automatic Logout**:
- 401 response triggers logout
- Session timeout handling
- Token refresh mechanism (if implemented)

**Security Best Practices**:
- No credentials stored in UserDefaults
- Secure token transmission
- Automatic session cleanup


---

[← Previous: Development Setup](development-setup.md) | [Back to Index](../README.md) | [Next: Coding Standards →](coding-standards.md)
