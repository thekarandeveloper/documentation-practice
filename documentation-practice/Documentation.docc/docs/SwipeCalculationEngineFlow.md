# Swipe iOS Calculation Engine Flow Analysis

## 🏗️ Architecture Overview

The calculation engine is built on a **shared Kotlin Multiplatform (KMM)** foundation with Swift UI wrappers, using the `Swipe_calculation_engine` namespace for all calculation operations.

### Core Components
- **SearchProductVM**: Main view model managing product selection and document-level calculations
- **EditSearchedProductSheet**: Individual product editing interface with real-time calculations
- **CreateBillVM**: Overall bill/document management
- **Shared KMM Engine**: Cross-platform calculation logic

## 📱 Product Addition Flow

### 1. Initial Product Search & Selection
**Primary Files:**
- `SearchProductScreen.swift` - Main search interface
- `SearchProductsList.swift` - Product list display
- `SearchProductListItem.swift` - Individual product items

**Flow:**
```
User Search Input
↓
SwipeSearchView (SearchProductScreen.swift:71)
↓
ProductSearchUIEvents.FetchInitialProducts
↓
Products displayed in SearchProductsList
↓
Individual products rendered via SearchProductListItem
```

### 2. Product Addition to Cart
**Key Files & Line References:**
- `SearchProductListItem.swift:450` - Serial batch products addition
- `SearchProductListItem.swift:646` - Regular products via `onProductModified` callback
- `SearchProductVM.swift:367-388` - `productModified()` method

**Addition Flow:**
```swift
// When quantity changes in SearchProductListItem (line 646)
onProductModified(modifiedProduct)
↓
// SearchProductVM.swift:367
SearchProductVM.productModified()
↓
// Update selectedProducts array (line 378 or 164)
selectedProducts.append(product) // or selectedProducts[index] = product
↓
// Triggers didSet observer (line 36)
selectedProducts didSet {
    calculateWithSelectedProducts()  // Line 39
    if tdsUnderGst {
        calculateWithSelectedProductsForTDSUnderGst()  // Line 41
    }
}
```

### 3. Document-Level Calculations
**File:** `SearchProductVM.swift:403-430`

**Core Calculation Method:**
```swift
func calculateWithSelectedProducts() {
    // 1. Reset and recalculate additional charges (lines 407-418)
    customAdditionalCharges.forEach { charge in
        resetEngineValues(charge: charge)
        let result = if withTax == 0 {
            calculateAdditionalChargeResult(additionalChargesInfo:
                Swipe_calculation_engineAdditionalChargesInfo(
                    documentNetAmount: documentCalculateNetAmount(),
                    netAmountPercentage: charge.percent,
                    withTaxAmount: charge.netAmount,
                    withoutTaxAmount: charge.netAmount,
                    tax: charge.tax,
                    withTax: 0,
                    type: charge.type
                ))
        } else {
            calculateAdditionalChargeResult(additionalChargesInfo:
                Swipe_calculation_engineAdditionalChargesInfo(
                    documentNetAmount: documentCalculateNetAmount(),
                    netAmountPercentage: charge.percent,
                    withTaxAmount: charge.totalAmount,
                    withoutTaxAmount: 0,
                    tax: charge.tax,
                    withTax: 1,
                    type: charge.type
                ))
        }
        charge.netAmount = result.totalAdditionalChargesWithoutTax
        charge.totalAmount = result.totalAdditionalChargesWithTax
        charge.percent = result.netAmountPercentage
        charge.taxAmount = result.taxAmount
    }

    // 2. Create comprehensive calculation request (line 424)
    let request = ProductSearchUIEvents.CalculateSelectedProductsDocumentResult(
        products: selectedProducts,
        discountType: selectedDiscountType.discountType,
        roundOffEnabled: shouldRoundOff,
        additionalCharges: customAdditionalCharges
            .filter({ $0.netAmount != 0 })
            .map({ $0.toEngineAdditionalCharge(documentNetAmount: documentCalculateNetAmount()) })
            .filter { $0.withTaxAmount != 0 },
        extraDiscount: extraDiscountAmount,
        withTax: Int32(withTax),
        tdsPercentage: selectedTds?.tax ?? 0.0,
        tdsAppliedOn: selectedTds?.tdsAppliedOn ?? .netAmount,
        tcsPercentage: selectedTcs?.tax ?? 0.0,
        tcsAppliedOn: applyTcsTaxOn,
        isRCMAmountAvailable: isRCMAmountAvailable,
        isCustomLinkingOn: isCustomLinkingOn,
        order: order,
        customColumnValues: customColumnsValuesMap,
        customColumnIdNameMap: customColumnsIdNameMap
    )

    // 3. Send to shared KMM engine (line 429)
    onUIEvent(events: request)
}
```

**Additional Triggers for Document Calculation:**
- `customAdditionalCharges` didSet (line 19)
- `extraDiscountAmount` didSet (line 27)
- `shouldRoundOff` didSet (line 97)
- `isRCMAmountAvailable` didSet (line 105)
- `didSelectTcs` didSet (line 111)
- `didSelectTds` didSet (line 160)
- `withTax` didSet (line 203)

## ✏️ Product Editing Flow (EditSearchedProductSheet)

### 1. Edit Sheet Initialization
**File:** `EditSearchedProductSheet.swift:303-383`

**OnAppear Setup:**
```swift
.onAppear(perform: {
    // Get product taxes for dynamic tax selection
    getProductTaxes()

    // Set up feature permissions for additional cess
    getFeaturePermissions(permissionKey: "additional_cess", accessKey: "additional_cess") { result in
        searchProductVM.accessToAdditionalCess = result
    }

    // Initialize settings for product details update
    settingsVM.setupProductDetailsUpdate()

    // Set engine values from product data
    engineUnitPrice = Double(unitPrice) ?? 0.0
    enginePriceWithTax = 0.0
    engineSelectedQuantity = Double(productQuantity) ?? 0.0
    engineTax = product.tax
    engineDiscountPercentage = Double(discountPercentage) ?? 0.0

    // Set product properties
    productSellingPrice = String(product.sellingPrice)
    productDiscount = String(product.productDiscount)
    dynamicProductTax = String(product.tax)

    // Handle additional cess
    if product.cess_on_qty_value != 0.0 {
        showACessTextField = true
        searchProductVM.additionalCessString = String(product.cess_on_qty_value)
    }

    // Set discount type
    discountValueIn = (product.is_discount_percent == 1) ? .PERCENTAGE : .CURRENCY
    freeQuantity = String(product.freeQty)

    // Setup custom columns (lines 329-352)
    customColumns = createBillVM.initialBillDetails.dataValue()?.productCustomColumns.columns.filter {
        $0.document_types.contains { $0.lowercased() == docType.identificationConstant.rawValue.lowercased() } ||
        $0.document_types.first?.lowercased() == "all"
    } ?? []

    // Initialize custom column values
    searchProductVM.customColumnsValuesMap = product.custom_col_values ?? [:]

    // Set product unit and conversion rates
    productUnit = product.unit ?? ""
    if product.conversionRate == 0 {
        product.currentConversionRate = 1
        product.previousConversionRate = 1
    } else {
        product.conversionCalculation = false
    }

    // Initial calculation
    calculate()  // Line 362

    // Auto-expand sections with existing values
    if discountAmount != "0.0" || discountPercentage != "0.0" {
        expandDiscountDetails = true
    }
})
```

### 2. Real-time Calculation Function
**File:** `EditSearchProduct+HelperFunctions.swift:16-18`

**Core Individual Product Calculation:**
```swift
func calculate() {
    searchProductVM.onUIEvent(events: ProductSearchUIEvents.CalculateIndividualProductResult(
        productInfo: Swipe_calculation_engineProductInfo(
            unitPrice: isPriceWithTax ? 0 : engineUnitPrice,
            selectedQuantity: engineSelectedQuantity,
            tax: engineTax,
            cess: product.cess,
            cessOnQty: product.cess_on_qty_value,
            discount: isDiscountAmount ? 0 : engineDiscountPercentage,
            discountType: searchProductVM.selectedDiscountType.discountType,
            previousConversionRate: product.previousConversionRate,
            currentConversionRate: product.currentConversionRate,
            conversionCalculation: product.conversionCalculation,
            withTax: Int32(searchProductVM.withTax),
            priceWithTax: isPriceWithTax ? enginePriceWithTax : 0,
            discountAmount: isDiscountAmount ? engineDiscountAmount : 0.0,
            isCustomLinkingOn: searchProductVM.isCustomLinkingOn,
            order: searchProductVM.order,
            customColumnValues: searchProductVM.customColumnsValuesMap,
            customColumnIdNameMap: searchProductVM.customColumnsIdNameMap
        )
    ))
}
```

### 3. Calculation Triggers in EditSearchedProductSheet
**Multiple trigger points throughout the file:**

1. **Custom Columns Change** (line 180):
   ```swift
   CustomColumnsView(customColumns: customColumns, onValueChanged: { map in
       if searchProductVM.isCustomLinkingOn {
           calculate()
       }
   })
   ```

2. **Quantity Change** (line 430):
   ```swift
   .onChange(of: productQuantity) { newValue in
       if product.cess_on_qty_value != 0.0 {
           // Reset additional cess and recalculate
           product.cess_on_qty_value = 0.0
           calculate()
       }
   }
   ```

3. **Dynamic Tax Rate Change** (line 487):
   ```swift
   DynamicTaxSelectionSheet(
       onItemSelected: { selectedTax in
           dynamicProductTax = "\(selectedTax)"
           product.tax = Double(dynamicProductTax) ?? 0.0
           engineTax = product.tax
           isPriceWithTax = true
           calculate()
       }
   )
   ```

4. **Initial Load** (line 362):
   ```swift
   .onAppear {
       // ... setup code ...
       calculate()
   }
   ```

### 4. Result Processing & UI Updates
**File:** `EditSearchedProductSheet.swift:384-422`

**Individual Product Result Handler:**
```swift
.onChange(of: searchProductVM.individualSelectedProductResult) { productResult in
    engineResult = productResult

    // Update unit price if not currently being edited
    if engineUnitPrice == 0 || !focusUnitPrice {
        unitPrice = String(productResult.unitPrice)
        engineUnitPrice = productResult.unitPrice
    }

    // Update price with tax if not currently being edited
    if enginePriceWithTax == 0 || !focusPriceWithTax {
        priceWithTax = String(productResult.priceWithTax)
        enginePriceWithTax = productResult.priceWithTax
    }

    // Update discount values based on discount type
    if discountValueIn == .CURRENCY {
        discountPercentage = String(productResult.discountPercentage)
        engineDiscountPercentage = productResult.discountPercentage

        if !isDiscountAmount {
            discountAmount = String(productResult.totalDiscount)
        }
    } else {
        discountAmount = String(productResult.totalDiscount)
    }

    // Handle custom linking calculations
    if searchProductVM.isCustomLinkingOn {
        if !focusSelectedQty {
            productQuantity = String(productResult.selectedQuantity)
        }
        searchProductVM.customColumnsValuesMap = productResult.customColumnMap

        // Update custom column UI values
        for column in customColumns {
            column.new_value = searchProductVM.customColumnsValuesMap[column.name] ?? ""
        }
    }

    isDiscountAmount = false

    // Logging
    loggerVM.addLog(CBLog(
        viewName: VIEW_NAME,
        event: .update(name: product.productName ?? ""),
        message: "Individual Product Result \(searchProductVM.individualSelectedProductResult)"
    ))
}
```

### 5. Product Update & Persistence Flow
**File:** `EditSearchProduct+HelperFunctions.swift:20-174`

**When user saves changes:**
```swift
// 1. Update calculated properties (lines 20-40)
func updateProductProperties() {
    product.selectedQty = Double(productQuantity) ?? 1.0
    product.totalPrice = searchProductVM.individualSelectedProductResult.totalAmount
    product.netAmount = KotlinDouble(value: searchProductVM.individualSelectedProductResult.netAmount)
    product.priceWithTax = searchProductVM.individualSelectedProductResult.priceWithTax
    product.totalDiscount = searchProductVM.individualSelectedProductResult.totalDiscount
    product.totalTax = searchProductVM.individualSelectedProductResult.totalTax
    product.unitPrice = searchProductVM.individualSelectedProductResult.unitPrice
    product.cessAmount = searchProductVM.individualSelectedProductResult.cessAmount
    product.discount = searchProductVM.individualSelectedProductResult.discountPercentage
    product.isSelected = true
    product.is_discount_percent = (discountValueIn == .PERCENTAGE) ? 1 : 0
    product.freeQty = Float(freeQuantity) ?? 0.0
    product.sellingPrice = Double(productSellingPrice) ?? 0.0
    product.productDiscount = Double(productDiscount) ?? 0.0
}

// 2. Update metadata (lines 42-57)
func updateCustomColumnsAndDescription() {
    // Update custom columns
    var map: [String: String] = [:]
    for (key, value) in searchProductVM.customColumnsValuesMap {
        map[key] = value
    }
    product.custom_col_values = map

    // Update description and conversion
    product.description_ = text
    product.conversionRate = product.currentConversionRate
    product.conversionCalculation = false

    // Clear custom columns map
    searchProductVM.customColumnsValuesMap = [:]
}

// 3. Persist to selected products (lines 152-174)
func updateProductInSelectedProducts() {
    let firstIndexInSelectedProducts = searchProductVM.selectedProducts.firstIndex(where: {
        ($0.productId == product.productId) &&
        ($0.variantName == product.variantName) &&
        ($0.copyCount == product.copyCount)
    })

    let firstIndexInAllProducts = searchProductVM.products.firstIndex(where: {
        ($0.productId == product.productId) &&
        ($0.variantName == product.variantName) &&
        ($0.copyCount == product.copyCount)
    })

    // Update in products array
    if let firstIndexInAllProducts = firstIndexInAllProducts {
        searchProductVM.products[firstIndexInAllProducts] = product
    }

    // Update or add to selected products
    guard let firstIndexInSelectedProducts = firstIndexInSelectedProducts else {
        searchProductVM.selectedProducts.append(product)  // Triggers document recalculation
        return
    }

    searchProductVM.selectedProducts[firstIndexInSelectedProducts] = product  // Triggers document recalculation
}
```

## 🔄 Calculation Engine Integration Points

### 1. Individual Product Calculations
**Engine Interface:** `Swipe_calculation_engineProductInfo` → `Swipe_calculation_engineProductResult`

**Usage Context:**
- Real-time editing in EditSearchedProductSheet
- Immediate feedback on price/tax/discount changes

**Key Parameters:**
- `unitPrice` / `priceWithTax` (mutually exclusive based on `isPriceWithTax`)
- `selectedQuantity`
- `tax` (tax rate percentage)
- `cess` / `cessOnQty` (cess calculations)
- `discount` / `discountAmount` (percentage vs fixed amount)
- `discountType` (how discount is applied)
- `conversionCalculation` (unit conversion handling)
- `customColumnValues` (for formula-based custom fields)

### 2. Document-Level Calculations
**Engine Interface:** `ProductSearchUIEvents.CalculateSelectedProductsDocumentResult`

**Usage Context:**
- Overall document totals calculation
- Triggered when selectedProducts array changes
- Includes all additional charges, discounts, and taxes

**Comprehensive Parameters:**
- `products`: Complete array of selected products
- `discountType`: Document-level discount application method
- `roundOffEnabled`: Whether to apply rounding
- `additionalCharges`: Extra charges (shipping, handling, etc.)
- `extraDiscount`: Document-level discount
- `withTax`: Tax inclusion flag (0/1)
- `tdsPercentage` / `tdsAppliedOn`: TDS (Tax Deducted at Source) handling
- `tcsPercentage` / `tcsAppliedOn`: TCS (Tax Collected at Source) handling
- `isRCMAmountAvailable`: Reverse Charge Mechanism
- `isCustomLinkingOn`: Custom column formula calculations
- `order`: Column calculation precedence
- `customColumnValues` / `customColumnIdNameMap`: Custom field mappings

### 3. Additional Charge Calculations
**Engine Interface:** `Swipe_calculation_engineAdditionalChargesInfo`

**Usage Context:**
- Both individual and document-level calculations
- Handles percentage-based and fixed-amount charges
- Tax calculations on additional charges

**Parameters:**
- `documentNetAmount`: Base amount for percentage calculations
- `netAmountPercentage`: Percentage rate (calculated by engine)
- `withTaxAmount` / `withoutTaxAmount`: Gross/net amounts
- `tax`: Tax rate on the charge
- `withTax`: Whether charge includes tax (0/1)
- `type`: Charge type identifier

### 4. Key State Management Patterns

**SearchProductVM State Management:**
```swift
// Core reactive property that triggers all document calculations
@Published var selectedProducts = [Product]() {
    didSet {
        calculateWithSelectedProducts()  // Document-level calculation
        if tdsUnderGst {
            calculateWithSelectedProductsForTDSUnderGst()  // Special TDS calculation
        }
    }
}

// Additional reactive properties
@Published var customAdditionalCharges: [InvoiceCustomAdditionalCharge] = [] {
    didSet {
        calculateWithSelectedProducts()  // Recalculate on charges change
    }
}

@Published var extraDiscountAmount: Double = 0.0 {
    didSet {
        calculateWithSelectedProducts()  // Recalculate on discount change
    }
}
```

**Focus State Management in EditSearchedProductSheet:**
```swift
// Prevent UI overwrites during user input
@FocusState var focusPriceWithTax: Bool
@FocusState var focusUnitPrice: Bool
@FocusState var focusSelectedQty: Bool

// In result handler:
if engineUnitPrice == 0 || !focusUnitPrice {
    unitPrice = String(productResult.unitPrice)  // Only update if not being edited
}
```

**Custom Column Linking:**
```swift
// Advanced calculation engine for formula-based custom fields
@Published var isCustomLinkingOn = false
@Published var order: [Swipe_calculation_engineColumn] = []
@Published var customColumnsIdNameMap: [KotlinInt: String] = [:]
@Published var customColumnsValuesMap: [String: String] = [:]

// Column dependency resolution
let orderResult = getOrder(columns: self.customColumns)
if orderResult.success {
    self.order = orderResult.order
    // Map column IDs to names for engine
    for column in self.customColumns {
        customColumnsIdNameMap[KotlinInt(int: column.id)] = column.name
    }
}
```

## 🎯 Performance Optimizations & Best Practices

### 1. Debounced Input Validation
**File:** `SearchProductListItem.swift:127-136`
```swift
.onChange(of: selectedProductQuantity) { newValue in
    if !newValue.isEmpty {
        validationWorkItem?.cancel()  // Cancel previous validation
        validationWorkItem = DispatchWorkItem {
            handleSelectedProductQtyChange(value: newValue)
            validateQuantity(newValue)
        }
        // 500ms delay before triggering calculation
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5, execute: workItem)
    }
}
```

### 2. Strategic Async Updates
**Pattern used throughout:**
```swift
DispatchQueue.main.async {
    // UI updates after calculation results
    self.selectedProducts = updatedProducts
}

DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
    // Delayed updates to prevent UI conflicts
    if selectedProductQuantity != formattedValue {
        selectedProductQuantity = formattedValue
    }
}
```

### 3. Calculation Result Caching
**Individual product results cached:**
```swift
@State var engineResult: Swipe_calculation_engineProductResult? = nil

// Results stored and reused to prevent unnecessary recalculations
.onChange(of: searchProductVM.individualSelectedProductResult) { productResult in
    self.engineResult = productResult
    // Update UI only when needed
}
```

### 4. Conditional Calculation Triggers
**Smart triggering based on context:**
```swift
// Only calculate if custom linking is enabled
if searchProductVM.isCustomLinkingOn {
    calculate()
}

// Only update UI if field is not currently focused
if !focusPriceWithTax {
    priceWithTax = String(productResult.priceWithTax)
}
```

### 5. Batch Operations for Performance
**Recalculate all selected products efficiently:**
```swift
func recalculateSelectedProducts() {
    DispatchQueue.main.async {
        // Remove products with zero quantity
        self.selectedProducts.removeAll(where: { $0.selectedQty == 0 })

        var tempList = [Product]()

        // Recalculate all products in batch
        self.selectedProducts.forEach { product in
            let calculation = product.calculation(
                discountType: self.selectedDiscountType.discountType,
                withTax: Int32(self.withTax),
                isCustomLinkingOn: self.isCustomLinkingOn,
                // ... other parameters
            )
            tempList.append(product)
        }

        // Single assignment to trigger calculation
        self.selectedProducts = tempList
    }
}
```

## 🚨 Error Handling & Validation

### 1. Quantity Validation
**File:** `SearchProductListItem.swift:583-608`
```swift
private func validateQuantity(_ qty: String) {
    if !isComingFromGroupSubProductList && !allowNegativeQty {
        if let qtyValue = Double(qty) {
            let totalQty = qtyValue + Double(product.freeQty)

            if totalQty > product.qty {
                // Show stock shortage warning
                withAnimation {
                    showStockInOptionView = true
                }
                // Auto-hide after timeout
                DispatchQueue.main.asyncAfter(deadline: .now() + turnFalseTime) {
                    withAnimation {
                        showStockInOptionView = false
                    }
                }
            }
        }
    }
}
```

### 2. Serial Number Validation
**File:** `EditSearchProduct+HelperFunctions.swift:131-150`
```swift
func validateDuplicateSerials() -> Bool {
    // Get all existing serial numbers for this product
    let allSerialsForProduct = searchProductVM.selectedProducts
        .filter {
            $0.productId == product.productId &&
            !($0.batchId == product.batchId &&
              $0.variantId == product.variantId &&
              $0.copyCount == product.copyCount)
        }
        .compactMap { $0.batchNo }
        .filter { !$0.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty }
        .map { $0.trimmingCharacters(in: .whitespacesAndNewlines).lowercased() }

    let newSerial = currentSerialNumber.trimmingCharacters(in: .whitespacesAndNewlines).lowercased()

    if !newSerial.isEmpty && allSerialsForProduct.contains(newSerial) {
        failureAlertMessage = "This serial number already exists for this product. Please enter a different serial number."
        showFailureAlert = true
        return false
    }
    return true
}
```

### 3. Custom Column Order Validation
**File:** `SearchProductVM.swift:283-314`
```swift
func setupCustomColumns(columns: [CustomColumnResponse.Column]) {
    if let linkedColumns = columns.filter({
        $0.field_type == getConstants().KEY_COLUMN_TYPE_VALUE_FORMULA &&
        $0.is_active == 1
    }).first {

        let orderResult = getOrder(columns: columns)

        if orderResult.success {
            self.order = orderResult.order
            isCustomLinkingOn = true
        } else {
            // Show error if column dependencies can't be resolved
            shouldShowError = true
            errorMessage = orderResult.message
            isCustomLinkingOn = false
        }
    }
}
```

## 🔍 Key File Structure Summary

```
CreateBillScreen/
├── CreateBillScreen.swift              # Main bill creation interface
├── CreateBillVM.swift                  # Bill-level view model and state
└── Products/Search/
    ├── SearchProductScreen.swift       # Product search interface
    ├── SearchProductVM.swift           # Product selection and document calculations
    ├── SearchProductListItem.swift     # Individual product item with quantity controls
    ├── EditSearchedProductSheet.swift  # Product editing interface
    ├── EditSearchProduct+HelperFunctions.swift  # Calculation and validation logic
    └── EditSearchProduct+Views.swift   # UI components for product editing
```

**Calculation Flow Summary:**
1. **Product Search** → `SearchProductScreen` → `SearchProductListItem`
2. **Add to Cart** → `SearchProductVM.productModified()` → `calculateWithSelectedProducts()`
3. **Edit Product** → `EditSearchedProductSheet` → `calculate()` → Individual product calculation
4. **Save Changes** → `updateProductInSelectedProducts()` → Document recalculation
5. **Final Result** → Updated totals and UI refresh

This architecture ensures real-time, accurate calculations while maintaining responsive UI performance through strategic state management, debouncing, and efficient calculation engine integration.
