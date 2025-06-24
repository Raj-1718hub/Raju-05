<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Cricket Gear Store</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background: #fefefe;
    }
    header {
      background: #283618;
      color: #fefae0;
      padding: 10px 20px;
    }
    header h1 { margin: 0; color: #dda15e; }
    nav a {
      color: #fefae0;
      margin: 0 10px;
      text-decoration: none;
    }
    .top-controls, .auth-section {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      align-items: center;
      gap: 10px;
      padding: 10px;
      background: #fefae0;
    }
    .top-controls input[type="text"] {
      padding: 8px;
      width: 200px;
    }
    .cart-btn, .toggle-btn {
      padding: 8px 12px;
      background-color: #bc6c25;
      color: white;
      border: none;
      cursor: pointer;
      border-radius: 4px;
    }
    .container {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
      gap: 20px;
      padding: 20px;
    }
    .card {
      border: 1px solid #ccc;
      padding: 10px;
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    .card img {
      width: 100%;
      height: 180px;
      object-fit: cover;
      border-radius: 6px;
      cursor: pointer;
    }
    .card-content h3 { margin: 10px 0 5px; color: #283618; }
    .card-content p { margin: 5px 0; color: #333; }
    .card-content button {
      margin: 5px 5px 5px 0;
      padding: 8px;
      background: #606c38;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    #cartModal, #loginModal {
      display: none;
      position: fixed;
      top: 10%;
      left: 50%;
      transform: translateX(-50%);
      background: white;
      padding: 20px;
      border: 1px solid #ccc;
      z-index: 1000;
      max-width: 400px;
      border-radius: 8px;
    }
    #cartItems, #cartTotal { margin: 10px 0; }
    #overlay {
      display: none;
      position: fixed;
      top: 0; left: 0; right: 0; bottom: 0;
      background: rgba(0,0,0,0.5);
      z-index: 999;
    }
  </style>
</head>
<body>
  <header>
    <h1>Cricket Gear Store</h1>
    <nav>
      <a href="#">Home</a>
      <a href="#products">Products</a>
    </nav>
  </header>

  <div class="auth-section">
    <span id="userInfo">Guest</span>
    <button class="cart-btn" onclick="toggleLogin()" id="loginBtn">Login</button>
    <button class="cart-btn" onclick="logoutUser()" id="logoutBtn" style="display:none">Logout</button>
  </div>

  <div class="top-controls">
    <input type="text" id="search" placeholder="Search cricket gear..." oninput="renderProducts(this.value)" />
    <button class="cart-btn" onclick="sortByPrice()">Sort by Price</button>
    <button class="cart-btn" onclick="toggleCart()">View Cart (<span id="cartCount">0</span>)</button>
  </div>

  <section class="container" id="products"></section>

  <div id="cartModal">
    <h3>Cart</h3>
    <div id="cartItems"></div>
    <p id="cartTotal"></p>
    <button onclick="checkout()">Checkout</button>
    <button onclick="closeCart()">Close</button>
  </div>

  <div id="loginModal">
    <h3>Login/Register</h3>
    <input type="text" id="username" placeholder="Enter username" /><br><br>
    <button onclick="loginUser()">Login</button>
    <button onclick="closeLogin()">Cancel</button>
  </div>

  <div id="overlay" onclick="closeCart(); closeLogin();"></div>

  <script>
    const products = [
      { name: "Cricket Bat", price: 1499, desc: "Premium Willow Bat", img: "Bat.jpg" },
      { name: "Cricket Ball", price: 299, desc: "Leather Match Ball", img: "Ball.jpg" },
      { name: "Batting Pads", price: 1199, desc: "Durable Pads", img: "Pads.jpg" },
      { name: "Cricket Helmet", price: 1799, desc: "Full Protection Helmet", img: "Helmet.jpg" },
      { name: "Batting Gloves", price: 799, desc: "Comfortable Gloves", img: "Glove.jpg" },
      { name: "Thigh Guard", price: 499, desc: "Flexible Guard", img: "Guard.jpg" },
      { name: "Cricket Jersey", price: 699, desc: "Polyester Jersey", img: "Jesrey.jpg" },
      { name: "Cricket Shoes", price: 1599, desc: "Anti-skid Shoes", img: "Shoe.jpg" },
      { name: "WK Gloves", price: 1299, desc: "Wicket Keeping Gloves", img: "vk.jpg" },
      { name: "Kit Bag", price: 999, desc: "Spacious Kit Bag", img: "Bag.jpg" }
    ];

    let cart = JSON.parse(localStorage.getItem('cart')) || [];
    let currentUser = localStorage.getItem('user') || null;
    let isSorted = false;

    function renderProducts(filter = "") {
      const container = document.getElementById('products');
      container.innerHTML = "";
      let list = [...products];
      if (isSorted) list.sort((a, b) => a.price - b.price);
      list = list.filter(p => p.name.toLowerCase().includes(filter.toLowerCase()));
      list.forEach(p => {
        container.innerHTML += `
        <div class="card">
          <img src="${p.img}" alt="${p.name}">
          <div class="card-content">
            <h3>${p.name}</h3>
            <p>${p.desc}</p>
            <p><strong>₹${p.price}</strong></p>
            <button onclick="addToCart('${p.name}')">Add to Cart</button>
            <button onclick="buyNow('${p.name}', ${p.price})">Buy Now</button>
          </div>
        </div>`;
      });
    }

    function buyNow(name, price) {
      if (!currentUser) {
        alert('Please login to buy products.');
        return;
      }
      alert(`Thank you ${currentUser} for buying ${name} at ₹${price}`);
    }

    function addToCart(name) {
      const item = products.find(p => p.name === name);
      const existing = cart.find(i => i.name === name);
      if (existing) existing.qty++;
      else cart.push({ ...item, qty: 1 });
      localStorage.setItem('cart', JSON.stringify(cart));
      updateCartCount();
      alert(`${name} added to cart`);
    }

    function updateCartCount() {
      document.getElementById('cartCount').innerText = cart.reduce((a, b) => a + b.qty, 0);
    }

    function toggleCart() {
      document.getElementById('cartModal').style.display = 'block';
      document.getElementById('overlay').style.display = 'block';
      renderCart();
    }

    function renderCart() {
      const cartItems = document.getElementById('cartItems');
      const total = cart.reduce((sum, item) => sum + item.price * item.qty, 0);
      cartItems.innerHTML = cart.map(item => `<p>${item.name} x ${item.qty} - ₹${item.price * item.qty}</p>`).join('');
      document.getElementById('cartTotal').innerText = `Total: ₹${total}`;
    }

    function closeCart() {
      document.getElementById('cartModal').style.display = 'none';
      document.getElementById('overlay').style.display = 'none';
    }

    function checkout() {
      if (!currentUser) return alert('Please login to proceed.');
      alert(`Order placed by ${currentUser}!`);
      cart = [];
      localStorage.removeItem('cart');
      updateCartCount();
      closeCart();
    }

    function sortByPrice() {
      isSorted = !isSorted;
      renderProducts(document.getElementById('search').value);
    }

    function toggleLogin() {
      document.getElementById('loginModal').style.display = 'block';
      document.getElementById('overlay').style.display = 'block';
    }

    function loginUser() {
      const username = document.getElementById('username').value;
      if (!username) return alert('Enter username');
      localStorage.setItem('user', username);
      currentUser = username;
      document.getElementById('userInfo').innerText = username;
      document.getElementById('loginBtn').style.display = 'none';
      document.getElementById('logoutBtn').style.display = 'inline-block';
      closeLogin();
    }

    function logoutUser() {
      localStorage.removeItem('user');
      currentUser = null;
      document.getElementById('userInfo').innerText = 'Guest';
      document.getElementById('loginBtn').style.display = 'inline-block';
      document.getElementById('logoutBtn').style.display = 'none';
    }

    function closeLogin() {
      document.getElementById('loginModal').style.display = 'none';
      document.getElementById('overlay').style.display = 'none';
    }

    document.addEventListener('DOMContentLoaded', () => {
      renderProducts();
      updateCartCount();
      if (currentUser) {
        document.getElementById('userInfo').innerText = currentUser;
        document.getElementById('loginBtn').style.display = 'none';
        document.getElementById('logoutBtn').style.display = 'inline-block';
      }
    });
  </script>
</body>
</html>