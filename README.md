# CoffeDev

☕ CoffeDev — Cardápio Digital Interativo para Cafeterias
CoffeDev é um aplicativo mobile desenvolvido com React Native + Expo, pensado para oferecer uma experiência moderna e intuitiva em cafeterias. O app permite aos usuários navegar por itens do cardápio, montar pedidos personalizados e visualizar o resumo com total calculado, tudo em uma interface elegante com foco em usabilidade e design responsivo.
🔥 Funcionalidades:
Autenticação de usuários (Login/Cadastro)
Navegação fluida entre telas com transições suaves
Listagem de produtos do cardápio
Adição e remoção de itens no pedido
Tela de resumo com cálculo do total
Interface com cores inspiradas em cafés especiais (vermelho, branco e tons terrosos)
Compatível com Android e iOS


code"\

import React, { useState } from 'react';
import { View, Text, TouchableOpacity, FlatList, TextInput, Animated, StyleSheet } from 'react-native';
import QRCode from 'react-native-qrcode-svg';

const products = [
  { id: '1', name: 'Café Expresso', price: 'R$ 5,00', description: 'Café forte e intenso.' },
  { id: '2', name: 'Capuccino', price: 'R$ 7,50', description: 'Café com espuma cremosa.' },
  { id: '3', name: 'Mocha', price: 'R$ 8,00', description: 'Café com chocolate e leite vaporizado.' },
  { id: '4', name: 'Latte', price: 'R$ 6,00', description: 'Café com leite vaporizado.' },
];

export default function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [isLogin, setIsLogin] = useState(true);
  const [email, setEmail] = useState('');
  const [senha, setSenha] = useState('');

  const [cart, setCart] = useState([]);
  const [discount, setDiscount] = useState('');
  const [cartAnim] = useState(new Animated.Value(1));
  const [currentScreen, setCurrentScreen] = useState('menu');
  const [paymentMethod, setPaymentMethod] = useState(null);

  const handleAddToCart = (item) => {
    const exists = cart.find(product => product.id === item.id);
    let newCart = exists ? cart.filter(product => product.id !== item.id) : [...cart, item];
    setCart(newCart);
    triggerCartAnimation();
  };

  const triggerCartAnimation = () => {
    Animated.sequence([
      Animated.timing(cartAnim, { toValue: 1.2, duration: 200, useNativeDriver: true }),
      Animated.spring(cartAnim, { toValue: 1, friction: 3, useNativeDriver: true }),
    ]).start();
  };

  const calcularTotal = () => {
    let total = 0;
    cart.forEach(item => {
      const preco = parseFloat(item.price.replace('R$', '').replace(',', '.'));
      total += preco;
    });
    let finalTotal = discount === 'DEV10' ? total * 0.9 : total;
    return finalTotal.toFixed(2).replace('.', ',');
  };

  const handleLogin = () => {
    // Simulação de login
    if (email && senha) setIsLoggedIn(true);
  };

  if (!isLoggedIn) {
    return (
      <View style={styles.container}>
        <Text style={styles.title}>☕ CoffeDev</Text>
        <Text style={styles.subtitle}>{isLogin ? 'Bem-vindo de volta!' : 'Crie sua conta'}</Text>

        <TextInput
          style={styles.input}
          placeholder="Email"
          placeholderTextColor="#aaa"
          value={email}
          onChangeText={setEmail}
        />

        <TextInput
          style={styles.input}
          placeholder="Senha"
          placeholderTextColor="#aaa"
          secureTextEntry
          value={senha}
          onChangeText={setSenha}
        />

        <TouchableOpacity style={styles.button} onPress={handleLogin}>
          <Text style={styles.buttonText}>{isLogin ? 'Entrar' : 'Cadastrar'}</Text>
        </TouchableOpacity>

        <TouchableOpacity onPress={() => setIsLogin(!isLogin)}>
          <Text style={styles.toggleText}>
            {isLogin ? 'Ainda não tem conta? Cadastre-se' : 'Já tem conta? Faça login'}
          </Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      {currentScreen === 'menu' ? (
        <>
          <Animated.Text style={[styles.title, { transform: [{ scale: cartAnim }] }]}>☕ CoffeDev</Animated.Text>

          <FlatList
            data={products}
            keyExtractor={item => item.id}
            renderItem={({ item }) => {
              const isInCart = cart.some(product => product.id === item.id);
              return (
                <Animated.View style={[styles.card, { opacity: isInCart ? 0.7 : 1 }]}>
                  <Text style={styles.productName}>{item.name}</Text>
                  <Text style={styles.description}>{item.description}</Text>
                  <Text style={styles.price}>{item.price}</Text>

                  <TouchableOpacity
                    style={[styles.button, isInCart ? styles.buttonSelected : {}]}
                    onPress={() => handleAddToCart(item)}
                  >
                    <Text style={styles.buttonText}>{isInCart ? 'Remover' : 'Adicionar'}</Text>
                  </TouchableOpacity>
                </Animated.View>
              );
            }}
          />

          <View style={styles.discountArea}>
            <TextInput
              placeholder="Cupom de Desconto"
              value={discount}
              onChangeText={setDiscount}
              style={styles.input}
              placeholderTextColor="#aaa"
            />
          </View>

          <Animated.View style={[styles.cartContainer, { transform: [{ scale: cartAnim }] }]}>
            <Text style={styles.cartText}>🛒 {cart.length} item(s) | Total: R$ {calcularTotal()}</Text>
            <TouchableOpacity style={styles.cartButton} onPress={() => setCurrentScreen('cart')}>
              <Text style={styles.cartButtonText}>Ver Carrinho</Text>
            </TouchableOpacity>
          </Animated.View>
        </>
      ) : currentScreen === 'cart' ? (
        <>
          <Text style={styles.title}>Resumo do Pedido</Text>
          {cart.length === 0 ? (
            <Text style={styles.emptyText}>Seu carrinho está vazio!</Text>
          ) : (
            <FlatList
              data={cart}
              keyExtractor={item => item.id}
              renderItem={({ item }) => (
                <View style={styles.cartItem}>
                  <Text style={styles.itemName}>{item.name}</Text>
                  <Text style={styles.itemPrice}>{item.price}</Text>
                  <TouchableOpacity
                    style={styles.removeButton}
                    onPress={() => setCart(cart.filter(i => i.id !== item.id))}
                  >
                    <Text style={styles.removeButtonText}>Remover</Text>
                  </TouchableOpacity>
                </View>
              )}
            />
          )}
          <Text style={styles.totalText}>Total: R$ {calcularTotal()}</Text>
          <TouchableOpacity style={styles.checkoutButton} onPress={() => setCurrentScreen('payment')}>
            <Text style={styles.checkoutButtonText}>Finalizar Pedido</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.backButton} onPress={() => setCurrentScreen('menu')}>
            <Text style={styles.backButtonText}>Voltar para o Cardápio</Text>
          </TouchableOpacity>
        </>
      ) : currentScreen === 'payment' ? (
        <>
          <Text style={styles.title}>Escolha a Forma de Pagamento</Text>
          <TouchableOpacity style={styles.paymentButton} onPress={() => { setPaymentMethod('pix'); setCurrentScreen('payment-confirmation'); }}>
            <Text style={styles.paymentButtonText}>Pagar com Pix</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.paymentButton} onPress={() => { setPaymentMethod('cartao'); setCurrentScreen('payment-confirmation'); }}>
            <Text style={styles.paymentButtonText}>Pagar com Cartão</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.backButton} onPress={() => setCurrentScreen('menu')}>
            <Text style={styles.backButtonText}>Voltar para o Cardápio</Text>
          </TouchableOpacity>
        </>
      ) : currentScreen === 'payment-confirmation' ? (
        <>
          <Text style={styles.title}>Confirmação do Pedido</Text>
          <Text style={styles.paymentConfirmationText}>Pedido confirmado! Aguardando pagamento...</Text>
          {paymentMethod === 'pix' && (
            <View style={styles.qrCodeContainer}>
              <QRCode value="https://www.exemplo.com/pix" size={200} />
              <Text style={styles.qrCodeText}>Escaneie o QR Code para pagar via Pix</Text>
            </View>
          )}
          {paymentMethod === 'cartao' && (
            <Text style={styles.paymentMethodText}>Escolheu pagar com Cartão. Insira os detalhes do seu cartão.</Text>
          )}
          <TouchableOpacity style={styles.backButton} onPress={() => setCurrentScreen('menu')}>
            <Text style={styles.backButtonText}>Voltar para o Cardápio</Text>
          </TouchableOpacity>
        </>
      ) : null}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#D9C5A4', padding: 20 },
  title: { fontSize: 28, color: '#4E342E', fontWeight: 'bold', marginBottom: 20, textAlign: 'center' },
  subtitle: { fontSize: 18, color: '#6D4C41', textAlign: 'center', marginBottom: 30 },
  input: { backgroundColor: '#F1E0C6', borderRadius: 8, padding: 12, marginBottom: 15, fontSize: 16, color: '#4E342E' },
  button: { backgroundColor: '#6D4C41', padding: 15, borderRadius: 8, alignItems: 'center', marginBottom: 20 },
  buttonText: { color: '#fff', fontWeight: 'bold', fontSize: 18 },
  toggleText: { color: '#4CAF50', textAlign: 'center', fontSize: 16 },
  card: { backgroundColor: '#F1E0C6', borderRadius: 8, padding: 15, marginVertical: 8 },
  productName: { fontSize: 18, fontWeight: 'bold', color: '#4E342E' },
  description: { fontSize: 14, color: '#7B5E57', marginTop: 5 },
  price: { fontSize: 16, color: '#6D4C41', marginTop: 10 },
  buttonSelected: { backgroundColor: '#4CAF50' },
  cartContainer: { marginTop: 20, backgroundColor: '#F1E0C6', padding: 15, borderRadius: 8 },
  cartText: { fontSize: 18, color: '#4E342E', fontWeight: 'bold' },
  cartButton: { backgroundColor: '#6D4C41', paddingVertical: 10, borderRadius: 8, marginTop: 15, alignItems: 'center' },
  cartButtonText: { color: '#fff', fontSize: 18, fontWeight: 'bold' },
  removeButton: { backgroundColor: '#b22222', padding: 5, borderRadius: 8, marginTop: 5 },
  removeButtonText: { color: '#fff', fontSize: 14, fontWeight: 'bold' },
  totalText: { fontSize: 20, fontWeight: 'bold', color: '#6D4C41', marginTop: 20 },
  checkoutButton: { backgroundColor: '#6D4C41', paddingVertical: 12, borderRadius: 8, marginTop: 20, alignItems: 'center' },
  checkoutButtonText: { color: '#fff', fontSize: 18, fontWeight: 'bold' },
  backButton: { marginTop: 20, alignItems: 'center' },
  backButtonText: { fontSize: 16, color: '#4CAF50' },
  paymentButton: { backgroundColor: '#6D4C41', paddingVertical: 12, borderRadius: 8, marginTop: 20, alignItems: 'center' },
  paymentButtonText: { color: '#fff', fontSize: 18, fontWeight: 'bold' },
  qrCodeContainer: { alignItems: 'center', marginTop: 30 },
  qrCodeText: { fontSize: 18, color: '#4E342E', marginTop: 12 },
  paymentConfirmationText: { fontSize: 18, textAlign: 'center', marginTop: 30, color: '#4E342E' },
  paymentMethodText: { fontSize: 18, textAlign: 'center', marginTop: 30, color: '#4E342E' }
});

