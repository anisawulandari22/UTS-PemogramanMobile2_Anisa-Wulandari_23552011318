# UTS-PemogramanMobile2_Anisa-Wulandari_23552011318

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
  final String image; // Bisa digunakan untuk menampilkan gambar es krim

  ProductModel({
    required this.id,
    required this.name,
    required this.price,
    required this.image,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'price': price,
      'image': image,
    };
  }

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
// Lokasi: lib/blocs/cart_cubit.dart

import 'package:flutter_bloc/flutter_bloc.dart';
import '../models/product_model.dart'; 

// --- CartItem menjadi Item Nota ---
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

// --- CartState menjadi TransactionState ---
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

  // Digunakan saat kasir menekan produk di menu
  void addToCart(ProductModel product) {
    final currentItems = List<CartItem>.from(state.items);
    final existingIndex = currentItems.indexWhere((item) => item.product.id == product.id);

    if (existingIndex != -1) {
      // Jika produk sudah ada, tingkatkan QTY-nya
      final existingItem = currentItems[existingIndex];
      currentItems[existingIndex] = existingItem.copyWith(quantity: existingItem.quantity + 1);
    } else {
      // Tambahkan item baru
      currentItems.add(CartItem(product: product, quantity: 1));
    }

    emit(CartState(items: currentItems));
  }

  // Menghapus baris produk dari nota
  void removeFromCart(ProductModel product) {
    final currentItems = List<CartItem>.from(state.items);
    currentItems.removeWhere((item) => item.product.id == product.id);
    emit(CartState(items: currentItems));
  }

  // Mengubah QTY di nota
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
  
  // Fungsi Checkout/Selesaikan Transaksi
  void clearCart() {
    // Di aplikasi kasir sungguhan, ini akan menyimpan transaksi ke database
    // lalu mengosongkan nota.
    emit(CartState(items: [])); 
  }
}
```

---

## **4. Membuat Custom Widget (TodoCard)**

**File:** `widgets/product_card.dart`

```dart
// Lokasi: lib/widgets/product_card.dart

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
    
    // Kita ubah Card menjadi GestureDetector atau InkWell agar lebih cocok sebagai Tombol Menu Kasir
    return InkWell(
      onTap: () {
        // Panggil addToCart saat menu diklik
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
            // Gambar Produk/Es Krim
            Expanded(
              child: Image.network(
                product.image,
                fit: BoxFit.cover,
                errorBuilder: (context, error, stackTrace) {
                  return const Center(child: Icon(Icons.icecream, size: 50, color: Colors.pink));
                },
              ),
            ),
            
            // Nama dan Harga
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

**Widget ini menggunakan Stack** untuk menampilkan label prioritas di atas card.

---

## **5. Halaman Utama (Home Page)**

**File:** `pages/todo_home_page.dart`

```dart
// Lokasi: lib/pages/cart_home_page.dart (Tampilan Utama Kasir)

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../models/product_model.dart';
import '../blocs/cart_cubit.dart';
import '../widgets/product_card.dart';
import 'cart_summary_page.dart'; // Halaman Nota/Checkout

class CartHomePage extends StatelessWidget {
  const CartHomePage({super.key});

  final List<ProductModel> menuEsKrim = const [
    ProductModel(id: 'E1', name: 'Vanilla', price: 15000, image: 'assets\images\vanila.jpeg'),
    ProductModel(id: 'E2', name: 'Coklat', price: 18000, image: 'assets\images\coklat.jpeg'),
    ProductModel(id: 'E3', name: 'Strawberry', price: 16000, image: 'assets\images\strawberry.jpeg'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Aplikasi Kasir Es Krim'),
      ),
      body: Row(
        children: [
          // 1. Panel Menu Produk (80% Lebar)
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
          
          // 2. Panel Nota / Ringkasan Keranjang (20% Lebar)
          const Expanded(
            flex: 2,
            child: VerticalDivider(width: 1),
          ),
          Expanded(
            flex: 2,
            // Menggunakan CartSummaryPage sebagai panel nota
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
// Lokasi: lib/pages/cart_summary_page.dart (Panel Nota Kasir)

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/cart_cubit.dart';

class CartSummaryPage extends StatelessWidget {
  final bool isDrawer; // Tambahkan flag untuk menentukan tampilan (Panel Kasir vs Halaman Penuh)
  
  const CartSummaryPage({Key? key, this.isDrawer = false}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final cartCubit = context.read<CartCubit>();
    
    // Hapus Scaffold jika digunakan sebagai Panel/Drawer
    final Widget content = BlocBuilder<CartCubit, CartState>(
      builder: (context, state) {
        if (state.items.isEmpty) {
          return const Center(
            child: Text(
              'Nota Kosong. Pilih menu di samping.',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 16, color: Colors.grey),
            ),
          );
        }

        return Column(
          children: <Widget>[
            // Judul Nota
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Text(
                'NOTA PENJUALAN',
                style: TextStyle(
                  fontSize: isDrawer ? 18 : 22,
                  fontWeight: FontWeight.bold,
                  color: Colors.blueGrey[800],
                ),
              ),
            ),
            const Divider(height: 1),
            
            // 1. Daftar Item
            Expanded(
              child: ListView.builder(
                itemCount: state.items.length,
                itemBuilder: (context, index) {
                  final item = state.items[index];
                  return ListTile(
                    title: Text(item.product.name, style: const TextStyle(fontSize: 14)),
                    subtitle: Text(
                      '${item.quantity} x ${item.product.price.toStringAsFixed(0)}',
                      style: const TextStyle(fontSize: 12),
                    ),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        // Tombol Kurangi Quantity
                        IconButton(
                          icon: const Icon(Icons.remove, size: 20),
                          onPressed: () {
                            final newQty = item.quantity - 1;
                            cartCubit.updateQuantity(item.product, newQty);
                          },
                        ),
                        Text('${item.quantity}', style: const TextStyle(fontWeight: FontWeight.bold)),
                        // Tombol Tambah Quantity
                        IconButton(
                          icon: const Icon(Icons.add, size: 20),
                          onPressed: () {
                            final newQty = item.quantity + 1;
                            cartCubit.updateQuantity(item.product, newQty);
                          },
                        ),
                      ],
                    ),
                  );
                },
              ),
            ),

            // Garis pemisah
            const Divider(),

            // 2. Total Item dan Total Harga (Summary Footer)
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Total Item:'),
                      Text(
                        '${cartCubit.getTotalItems()} pcs',
                        style: const TextStyle(fontWeight: FontWeight.bold),
                      ),
                    ],
                  ),
                  const SizedBox(height: 8),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text(
                        'TOTAL HARGA:',
                        style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                      ),
                      Text(
                        'Rp ${cartCubit.getTotalPrice().toStringAsFixed(0)}',
                        style: TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.w900,
                          color: Colors.red[700],
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 20),

                  // 3. Tombol Checkout
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        padding: const EdgeInsets.symmetric(vertical: 15),
                        backgroundColor: Colors.blue,
                        foregroundColor: Colors.white,
                      ),
                      onPressed: () {
                        if (state.items.isEmpty) {
                           ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(content: Text('Nota masih kosong!')),
                          );
                          return;
                        }
                        // Memanggil fungsi clearCart() untuk menyelesaikan transaksi
                        cartCubit.clearCart();
                        
                        ScaffoldMessenger.of(context).showSnackBar(
                          const SnackBar(
                            content: Text('Transaksi Selesai! Nota dikosongkan.'),
                            duration: Duration(seconds: 2),
                          ),
                        );
                      },
                      child: const Text(
                        'BAYAR / SELESAIKAN TRANSAKSI',
                        style: TextStyle(fontSize: 16),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ],
        );
      },
    );

    // Jika dipanggil sebagai halaman penuh (misal di HP), gunakan Scaffold
    if (isDrawer) {
      return content;
    } else {
      return Scaffold(
        appBar: AppBar(title: const Text('Nota Penjualan')),
        body: content,
      );
    }
  }
}
```

---

## **7. Halaman GridView**

**File:** `pages/todo_grid_page.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/todo_cubit.dart';
import '../models/todo_model.dart';
import '../widgets/todo_card.dart';
import 'todo_detail_page.dart';

class TodoGridPage extends StatelessWidget {
  const TodoGridPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Grid View')),
      body: BlocBuilder<TodoCubit, List<Todo>>(
        builder: (context, todos) {
          if (todos.isEmpty) {
            return const Center(child: Text('Tidak ada tugas'));
          }
          return GridView.builder(
            padding: const EdgeInsets.all(12),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2, childAspectRatio: 1.1,
            ),
            itemCount: todos.length,
            itemBuilder: (context, index) {
              final todo = todos[index];
              return TodoCard(
                todo: todo,
                onTap: () => Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (_) => TodoDetailPage(todo: todo),
                  ),
                ),
                onDelete: () =>
                    context.read<TodoCubit>().removeTodo(todo.id),
                onToggle: () =>
                    context.read<TodoCubit>().toggleTodo(todo.id),
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
import 'blocs/todo_cubit.dart';
import 'pages/todo_home_page.dart';

void main() => runApp(const AdvancedTodoApp());

class AdvancedTodoApp extends StatelessWidget {
  const AdvancedTodoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => TodoCubit(),
      child: MaterialApp(
        debugShowCheckedModeBanner: false,
        title: 'Advanced Flutter BLoC To-Do',
        theme: ThemeData(
          useMaterial3: true,
          colorSchemeSeed: Colors.blueAccent,
        ),
        home: const TodoHomePage(),
      ),
    );
  }
}
```
   
