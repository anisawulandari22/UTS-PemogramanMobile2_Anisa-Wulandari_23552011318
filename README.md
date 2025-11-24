# Anisa-Wulandari
# 23552011318
# TIF RP-23 CNS A

## **Bagian A - Teori**
1. Jelaskan perbedaan antara Cubit dan Bloc dalam arsitektur Flutter.
2. Mengapa penting untuk memisahkan antara model data, logika bisnis, dan UI dalam pengembangan aplikasi Flutter?
3. Sebutkan dan jelaskan minimal tiga state yang mungkin digunakan dalam CrtCubit beserta fungsinya.
## **Jawab**
1. Baik Cubit maupun Bloc dari paket flutter_bloc adalah Business Logic Components (BLC) yang bertugas mengelola state menggunakan Stream.
   Cubit:
   - Sederhana & Ringkas.
   - Inputnya adalah Fungsi/Metode (misalnya, increment()).
   - Fungsi ini langsung memanggil emit() untuk memancarkan state baru.
   - Cocok untuk logika bisnis yang sederhana (misalnya, counter, toggle).
   Bloc:
   - Lebih Terstruktur.
   - Inputnya adalah Events (misalnya, IncrementEvent).
   - Membutuhkan pemetaan (mapEventToState) untuk menentukan state mana yang akan dipancarkan sebagai respons terhadap Event tertentu.
   - Cocok untuk logika bisnis yang kompleks dan membutuhkan reaksi terhadap Event spesifik (misalnya, Login dengan validasi bertahap).
  2. Memisahkan antara model data, logika bisnis, dan UI dalam pengembangan aplikasi Flutter penting karena Cubit adalah pola manajemen state Flutter yang sederhana, di mana fungsi secara langsung memancarkan State baru, cocok untuk logika ringan; sementara Bloc lebih terstruktur, membutuhkan Events yang dipetakan ke State, ideal untuk logika yang lebih kompleks. Pemisahan antara UI, Logika Bisnis (Cubit/Bloc), dan Data Model sangat penting untuk memastikan kemudahan pengujian (testability), pemeliharaan (maintainability) yang lebih baik, dan skalabilitas aplikasi, karena perubahan pada satu lapisan tidak merusak lapisan lain. Dalam implementasi CrtCubit (Keranjang Belanja), state yang digunakan berfungsi sebagai penanda kondisi saat ini, contohnya CartInitial (awal/kosong), CartLoading (sedang memuat data), CartLoaded (data berhasil ditampilkan), dan CartError (terjadi kegagalan).
3. State 1: CartInitial (State Awal)
Fungsi: Menandakan state awal atau kosong dari keranjang belanja.
Deskripsi: Cubit baru saja dibuat atau keranjang belanja telah dikosongkan. UI akan menampilkan indikator bahwa keranjang siap dimuat, atau mungkin menampilkan pesan "Keranjang kosong".
Variabel Data: Biasanya hanya memiliki daftar item keranjang yang kosong, misalnya List<CartItem> items = const [].
State 2: CartLoading (State Memuat Data)
Fungsi: Menandakan bahwa proses asinkron (misalnya, memuat item keranjang dari database atau API) sedang berlangsung.
Deskripsi: Digunakan saat aplikasi sedang mengambil data. UI harus menampilkan indikator pemuatan (misalnya, CircularProgressIndicator) untuk memberikan umpan balik kepada pengguna.
Variabel Data: Mungkin tidak memiliki data spesifik yang relevan selain penanda loading.
State 3: CartLoaded (State Data Berhasil Dimuat)
Fungsi: Menandakan bahwa data keranjang belanja telah berhasil dimuat atau diperbarui.
Deskripsi: Ini adalah state utama di mana UI menampilkan isi keranjang, total harga, dan tombol checkout.
Variabel Data: Harus menyimpan data keranjang yang relevan, seperti List<CartItem> items, double totalPrice, dan int totalItemCount.

## **2. Model Data Lengkap**

**File:** `models/product_model.dart`

```dart
class ProductModel {
  final String id;
  final String name;
  final int price;
  final String image;

  // PERBAIKAN: Tambahkan const pada konstruktor untuk efisiensi memori (Konstanta runtime)
  const ProductModel({
    required this.id,
    required this.name,
    required this.price,
    required this.image,
  });

  // Metode untuk konversi ke Map (untuk penyimpanan/database/API)
  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'price': price,
      'image': image,
    };
  }

  // Factory constructor untuk konversi dari Map (dari penyimpanan/database/API)
  factory ProductModel.fromMap(Map<String, dynamic> map) {
    return ProductModel(
      id: map['id'] as String,
      name: map['name'] as String,
      price: map['price'] as int,
      image: map['image'] as String,
    );
  }
}
```

---

## **3. Logika Bisnis (Cubit)**

**File:** `blocs/cart_cubit.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../models/product_model.dart'; 

class CartItem {
  final ProductModel product;
  final int quantity;

  CartItem({required this.product, required this.quantity});

  CartItem copyWith({int? quantity}) {
    return CartItem(
      product: product,
      quantity: quantity ?? this.quantity,
    );
  }
}

class CartState {
  final List<CartItem> items;

  CartState({required this.items});
}

class CartCubit extends Cubit<CartState> {
  CartCubit() : super(CartState(items: []));

  int getTotalItems() {
    return state.items.fold(0, (sum, item) => sum + item.quantity);
  }

  double getTotalPrice() {
    return state.items.fold(
      0.0,
      (sum, item) => sum + (item.product.price * item.quantity),
    );
  }

  void addToCart(ProductModel product) {
    final currentItems = List<CartItem>.from(state.items);
    final existingIndex = currentItems.indexWhere((item) => item.product.id == product.id);

    if (existingIndex != -1) {
      final existingItem = currentItems[existingIndex];
      currentItems[existingIndex] = existingItem.copyWith(quantity: existingItem.quantity + 1);
    } else {
      currentItems.add(CartItem(product: product, quantity: 1));
    }

    emit(CartState(items: currentItems));
  }

  void removeFromCart(ProductModel product) {
    final currentItems = List<CartItem>.from(state.items);
    currentItems.removeWhere((item) => item.product.id == product.id);
    emit(CartState(items: currentItems));
  }

  void updateQuantity(ProductModel product, int qty) {
    if (qty < 0) return;

    final currentItems = List<CartItem>.from(state.items);
    final existingIndex = currentItems.indexWhere((item) => item.product.id == product.id);

    if (existingIndex != -1) {
      if (qty == 0) {
        currentItems.removeAt(existingIndex);
      } else {
        final existingItem = currentItems[existingIndex];
        currentItems[existingIndex] = existingItem.copyWith(quantity: qty);
      }
    } 

    emit(CartState(items: currentItems));
  }
  
  void clearCart() {
    emit(CartState(items: [])); 
  }
}
```

---

## **4. Membuat Custom Widget (TodoCard)**

**File:** `widgets/product_card.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../models/product_model.dart';
import '../blocs/cart_cubit.dart';

class ProductCard extends StatelessWidget {
  final ProductModel product;
  
  const ProductCard({
    Key? key,
    required this.product,
    }) : super(key: key);
    
    @override
    Widget build(BuildContext context) {
      final cartCubit = context.read<CartCubit>();
      
      return InkWell(
        onTap: () {
          cartCubit.addToCart(product);
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('${product.name} ditambahkan ke nota!'),
              duration: const Duration(milliseconds: 500),
            ),
          );
        },
        child: Card(
          elevation: 4,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: <Widget>[
              Expanded(
                child: Image.asset( 
                product.image,
                fit: BoxFit.cover,
                errorBuilder: (context, error, stackTrace) {
                  return const Center(child: Icon(Icons.icecream, size: 50, color: Colors.pink));
                  },
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                      product.name,
                      style: const TextStyle(
                        fontSize: 14,
                        fontWeight: FontWeight.bold,
                      ),
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                      ),
                      const SizedBox(height: 4),
                      Text(
                        'Rp ${product.price.toStringAsFixed(0)}',
                        style: TextStyle(
                          fontSize: 12,
                          color: Colors.green[700],
                          fontWeight: FontWeight.w600,
                        ),
                      ),
                    ],
                ),
              ),
            ],
          ),
        ),
      );
   }
}
```

---

## **5. Halaman Utama (Home Page)**

**File:** `pages/card_home_page.dart`

```dart
import 'package:flutter/material.dart';
import '../models/product_model.dart';
import '../blocs/cart_cubit.dart';
import '../widgets/product_card.dart';
import 'cart_summary_page.dart';

class CartHomePage extends StatelessWidget {
  const CartHomePage({super.key});

  final List<ProductModel> menuEsKrim = const [
    ProductModel(id: 'E1', name: 'Vanilla Blast', price: 15000, image: 'assets/images/vanila.jpeg'),
    ProductModel(id: 'E2', name: 'Coklat Fudge', price: 18000, image: 'assets/images/coklat.jpeg'),
    ProductModel(id: 'E3', name: 'Strawberry Delight', price: 16000, image: 'assets/images/stawberry.jpeg'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Aplikasi Kasir Es Krim'),
      ),
      body: Row(
        children: [
          Expanded(
            flex: 3,
            child: GridView.builder(
              padding: const EdgeInsets.all(10),
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 3, 
                childAspectRatio: 0.8,
                crossAxisSpacing: 10,
                mainAxisSpacing: 10,
              ),
              itemCount: menuEsKrim.length,
              itemBuilder: (context, index) {
                return ProductCard(product: menuEsKrim[index]);
              },
            ),
          ),
          
          const VerticalDivider(width: 1),
          
          Expanded(
            flex: 2,
            child: CartSummaryPage(isDrawer: true),
          ),
        ],
      ),
    );
  }
}
```

---

## **6. Halaman Detail**

**File:** `pages/card_summary_page.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/cart_cubit.dart';

class CartSummaryPage extends StatelessWidget {
  final bool isDrawer;

  const CartSummaryPage({super.key, this.isDrawer = false});

  @override
  Widget build(BuildContext context) {
    final cartCubit = context.read<CartCubit>();

    final Widget content = BlocBuilder<CartCubit, CartState>(
      builder: (context, state) {
        if (state.items.isEmpty) {
          return const Center(
            child: Text(
              'Keranjang/Nota Kosong.',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 16, color: Colors.grey),
            ),
          );
        }

        return Column(
          children: <Widget>[
            Expanded(
              child: ListView.builder(
                itemCount: state.items.length,
                itemBuilder: (context, index) {
                  final item = state.items[index];
                  return ListTile(
                    title: Text(item.product.name, style: const TextStyle(fontSize: 14)),
                    subtitle: Text(
                      '${item.quantity} x Rp ${item.product.price.toStringAsFixed(0)}',
                      style: const TextStyle(fontSize: 12),
                    ),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        IconButton(
                          icon: const Icon(Icons.remove, size: 20),
                          onPressed: () => cartCubit.updateQuantity(item.product, item.quantity - 1),
                        ),
                        Text('${item.quantity}', style: const TextStyle(fontWeight: FontWeight.bold)),
                        IconButton(
                          icon: const Icon(Icons.add, size: 20),
                          onPressed: () => cartCubit.updateQuantity(item.product, item.quantity + 1),
                        ),
                      ],
                    ),
                  );
                },
              ),
            ),
            
            const Divider(),

            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Total Item:'),
                      Text('${cartCubit.getTotalItems()} pcs', style: const TextStyle(fontWeight: FontWeight.bold)),
                    ],
                  ),
                  const SizedBox(height: 8),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('TOTAL HARGA:', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                      Text(
                        'Rp ${cartCubit.getTotalPrice().toStringAsFixed(0)}',
                        style: TextStyle(fontSize: 18, fontWeight: FontWeight.w900, color: Colors.red[700]),
                      ),
                    ],
                  ),
                  const SizedBox(height: 20),

                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: () {
                        if (state.items.isEmpty) return;
                        cartCubit.clearCart();
                        ScaffoldMessenger.of(context).showSnackBar(
                          const SnackBar(content: Text('Checkout Berhasil! Nota dikosongkan.')),
                        );
                      },
                      child: const Text('CHECKOUT'),
                    ),
                  ),
                ],
              ),
            ),
          ],
        );
      },
    );

    if (isDrawer) {
      return Container(color: Colors.white, child: content);
    } else {
      return Scaffold(
        appBar: AppBar(title: const Text('Ringkasan Keranjang')),
        body: content,
      );
    }
  }
}
```

---

## **7. Halaman GridView**

**File:** `pages/cart_grid_page.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/cart_cubit.dart';
import '../models/product_model.dart';
import '../widgets/product_card.dart';
import 'cart_home_page.dart';

class CartGridPage extends StatelessWidget {
  const CartGridPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Product Grid View')),
      body: BlocBuilder<CartCubit, CartState>(
        builder: (context, state) {
          final cartItems = state.items;
          if (cartItems.isEmpty) {
            return const Center(child: Text('Keranjang kosong.'));
          }
          return GridView.builder(
            padding: const EdgeInsets.all(12),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2, 
              childAspectRatio: 1.1,
            ),
            itemCount: cartItems.length,
            itemBuilder: (context, index) {
              final cartItem = cartItems[index];
              final product = cartItem.product;
              
              return ProductCard(
                product: product,             
              );
            },
          );
        },
      ),
    );
  }
}
```

---

## **8. Main Entry**

**File:** `main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'blocs/cart_cubit.dart'; 
import 'pages/cart_home_page.dart'; 

void main() {
  runApp(const CashierApp());
}

class CashierApp extends StatelessWidget {
  const CashierApp({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => CartCubit(),
      
      child: MaterialApp(
        title: 'Aplikasi Kasir Es Krim',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.pinkAccent), 
          useMaterial3: true,
          appBarTheme: const AppBarTheme(
            backgroundColor: Colors.pinkAccent,
            foregroundColor: Colors.white,
          ),
        ),
        home: CartHomePage(),
        debugShowCheckedModeBanner: false,
      ),
    );
  }
}
```
   
