<template>
  <main class="order-page">
    <header class="order-hero">
      <div>
        <p class="eyebrow">SELF-SERVICE ORDER</p>
        <h1>简单点菜</h1>
        <p>搜索喜欢的菜品，加入订单后即可确认下单。</p>
      </div>
      <div class="cart-summary">
        <span>已选 {{ totalCount }} 份</span>
        <strong>¥{{ totalPrice }}</strong>
      </div>
    </header>

    <section class="toolbar" aria-label="菜单筛选">
      <label class="search-box">
        <span>🔍</span>
        <input v-model.trim="keyword" type="search" placeholder="搜索菜名或介绍" />
      </label>
      <div class="categories">
        <button
          v-for="category in categories"
          :key="category"
          :class="{ active: activeCategory === category }"
          type="button"
          @click="activeCategory = category"
        >
          {{ category }}
        </button>
      </div>
    </section>

    <div class="content-grid">
      <section>
        <div class="section-title">
          <h2>今日菜单</h2>
          <span>{{ filteredDishes.length }} 道菜</span>
        </div>

        <div v-if="filteredDishes.length" class="dish-grid">
          <article v-for="dish in filteredDishes" :key="dish.id" class="dish-card">
            <div class="dish-emoji">{{ dish.emoji }}</div>
            <div class="dish-body">
              <div class="dish-heading">
                <div>
                  <span class="category-tag">{{ dish.category }}</span>
                  <h3>{{ dish.name }}</h3>
                </div>
                <strong>¥{{ dish.price }}</strong>
              </div>
              <p>{{ dish.description }}</p>
              <button type="button" @click="addDish(dish)">加入订单</button>
            </div>
          </article>
        </div>
        <div v-else class="empty-state">没有找到符合条件的菜品。</div>
      </section>

      <aside class="order-panel">
        <div class="section-title">
          <h2>我的订单</h2>
          <button v-if="cart.length" class="text-button" type="button" @click="clearCart">清空</button>
        </div>

        <div v-if="cart.length" class="cart-list">
          <div v-for="item in cart" :key="item.id" class="cart-item">
            <div>
              <strong>{{ item.name }}</strong>
              <small>¥{{ item.price }} / 份</small>
            </div>
            <div class="quantity">
              <button type="button" :aria-label="`减少${item.name}`" @click="changeQuantity(item.id, -1)">−</button>
              <span>{{ item.quantity }}</span>
              <button type="button" :aria-label="`增加${item.name}`" @click="changeQuantity(item.id, 1)">＋</button>
            </div>
          </div>

          <div class="total-row">
            <span>合计</span>
            <strong>¥{{ totalPrice }}</strong>
          </div>
          <button class="submit-button" type="button" @click="submitOrder">确认下单</button>
        </div>
        <div v-else class="empty-cart">
          <span>🧾</span>
          <p>还没有选择菜品</p>
        </div>
      </aside>
    </div>
  </main>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

interface Dish {
  id: number
  name: string
  category: string
  description: string
  price: number
  emoji: string
}

interface CartItem extends Dish {
  quantity: number
}

const dishes: Dish[] = [
  { id: 1, name: '宫保鸡丁', category: '热菜', description: '鸡肉、花生与时蔬，香辣微甜。', price: 28, emoji: '🍗' },
  { id: 2, name: '番茄炒蛋', category: '热菜', description: '家常风味，酸甜开胃。', price: 18, emoji: '🍅' },
  { id: 3, name: '牛肉面', category: '主食', description: '慢炖牛肉配劲道面条。', price: 26, emoji: '🍜' },
  { id: 4, name: '扬州炒饭', category: '主食', description: '米饭粒粒分明，配料丰富。', price: 20, emoji: '🍚' },
  { id: 5, name: '拍黄瓜', category: '凉菜', description: '清脆爽口，蒜香十足。', price: 12, emoji: '🥒' },
  { id: 6, name: '水果沙拉', category: '凉菜', description: '当季水果搭配清爽酸奶。', price: 16, emoji: '🥗' },
  { id: 7, name: '冰柠檬茶', category: '饮品', description: '现切柠檬，清凉解腻。', price: 10, emoji: '🍋' },
  { id: 8, name: '热豆浆', category: '饮品', description: '香浓顺滑，现磨无负担。', price: 8, emoji: '🥛' },
]

const categories = ['全部', ...new Set(dishes.map((dish) => dish.category))]
const keyword = ref('')
const activeCategory = ref('全部')
const cart = ref<CartItem[]>([])

const filteredDishes = computed(() => {
  const normalizedKeyword = keyword.value.toLowerCase()
  return dishes.filter((dish) => {
    const matchesCategory = activeCategory.value === '全部' || dish.category === activeCategory.value
    const matchesKeyword =
      !normalizedKeyword ||
      dish.name.toLowerCase().includes(normalizedKeyword) ||
      dish.description.toLowerCase().includes(normalizedKeyword)
    return matchesCategory && matchesKeyword
  })
})

const totalCount = computed(() => cart.value.reduce((sum, item) => sum + item.quantity, 0))
const totalPrice = computed(() =>
  cart.value.reduce((sum, item) => sum + item.price * item.quantity, 0),
)

function addDish(dish: Dish) {
  const existingItem = cart.value.find((item) => item.id === dish.id)
  if (existingItem) {
    existingItem.quantity += 1
  } else {
    cart.value.push({ ...dish, quantity: 1 })
  }
  ElMessage.success(`已添加：${dish.name}`)
}

function changeQuantity(id: number, amount: number) {
  const item = cart.value.find((cartItem) => cartItem.id === id)
  if (!item) return

  item.quantity += amount
  if (item.quantity <= 0) {
    cart.value = cart.value.filter((cartItem) => cartItem.id !== id)
  }
}

function clearCart() {
  cart.value = []
}

async function submitOrder() {
  try {
    await ElMessageBox.confirm(
      `共 ${totalCount.value} 份，合计 ¥${totalPrice.value}。确认下单吗？`,
      '确认订单',
      { confirmButtonText: '确认', cancelButtonText: '再看看', type: 'success' },
    )
    cart.value = []
    ElMessage.success('下单成功！')
  } catch {
    // User cancelled the confirmation dialog.
  }
}
</script>

<style scoped>
.order-page {
  width: min(1400px, calc(100% - 40px));
  margin: 0 auto;
  padding: 32px 0 56px;
  color: #243047;
}

.order-hero {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 24px;
  padding: 34px;
  border-radius: 24px;
  color: white;
  background: linear-gradient(135deg, #ff7a45, #f04438);
  box-shadow: 0 18px 40px rgb(240 68 56 / 20%);
}

.eyebrow {
  margin: 0 0 6px;
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 0.18em;
  opacity: 0.8;
}

.order-hero h1 {
  margin: 0;
  font-size: clamp(30px, 4vw, 46px);
}

.order-hero p:last-child {
  margin: 10px 0 0;
  opacity: 0.9;
}

.cart-summary {
  min-width: 150px;
  padding: 16px 20px;
  border: 1px solid rgb(255 255 255 / 28%);
  border-radius: 16px;
  background: rgb(255 255 255 / 14%);
  text-align: right;
}

.cart-summary span,
.cart-summary strong {
  display: block;
}

.cart-summary strong {
  margin-top: 4px;
  font-size: 27px;
}

.toolbar {
  margin: 24px 0;
  padding: 18px;
  border: 1px solid #e8ecf3;
  border-radius: 18px;
  background: white;
}

.search-box {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 14px;
  border: 1px solid #d9dee8;
  border-radius: 12px;
}

.search-box:focus-within {
  border-color: #f04438;
  box-shadow: 0 0 0 3px rgb(240 68 56 / 10%);
}

.search-box input {
  width: 100%;
  padding: 13px 0;
  border: 0;
  outline: 0;
  color: #243047;
  background: transparent;
}

.categories {
  display: flex;
  flex-wrap: wrap;
  gap: 9px;
  margin-top: 14px;
}

.categories button,
.dish-body button,
.quantity button,
.submit-button,
.text-button {
  cursor: pointer;
}

.categories button {
  padding: 8px 15px;
  border: 0;
  border-radius: 999px;
  color: #667085;
  background: #f2f4f7;
}

.categories button.active {
  color: white;
  background: #f04438;
}

.content-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 340px;
  gap: 24px;
  align-items: start;
}

.section-title {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 14px;
}

.section-title h2 {
  margin: 0;
  font-size: 21px;
}

.section-title span {
  color: #98a2b3;
}

.dish-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 16px;
}

.dish-card {
  display: grid;
  grid-template-columns: 92px minmax(0, 1fr);
  overflow: hidden;
  border: 1px solid #e8ecf3;
  border-radius: 18px;
  background: white;
  transition: transform 0.2s, box-shadow 0.2s;
}

.dish-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 30px rgb(36 48 71 / 10%);
}

.dish-emoji {
  display: grid;
  place-items: center;
  min-height: 158px;
  font-size: 48px;
  background: #fff6f2;
}

.dish-body {
  padding: 18px;
}

.dish-heading {
  display: flex;
  justify-content: space-between;
  gap: 12px;
}

.dish-heading h3 {
  margin: 5px 0 0;
  font-size: 18px;
}

.dish-heading strong {
  color: #f04438;
}

.category-tag {
  color: #f04438;
  font-size: 12px;
  font-weight: 700;
}

.dish-body p {
  min-height: 42px;
  margin: 10px 0 14px;
  color: #667085;
  font-size: 14px;
  line-height: 1.5;
}

.dish-body > button,
.submit-button {
  width: 100%;
  padding: 10px 14px;
  border: 0;
  border-radius: 10px;
  color: white;
  font-weight: 700;
  background: #f04438;
}

.dish-body > button:hover,
.submit-button:hover {
  background: #d92d20;
}

.order-panel {
  position: sticky;
  top: 20px;
  padding: 22px;
  border: 1px solid #e8ecf3;
  border-radius: 18px;
  background: white;
}

.text-button {
  border: 0;
  color: #f04438;
  background: transparent;
}

.cart-list {
  display: grid;
  gap: 12px;
}

.cart-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding-bottom: 12px;
  border-bottom: 1px solid #eef0f4;
}

.cart-item strong,
.cart-item small {
  display: block;
}

.cart-item small {
  margin-top: 4px;
  color: #98a2b3;
}

.quantity {
  display: flex;
  align-items: center;
  gap: 9px;
}

.quantity button {
  width: 28px;
  height: 28px;
  border: 1px solid #d9dee8;
  border-radius: 8px;
  color: #344054;
  background: white;
}

.total-row {
  display: flex;
  justify-content: space-between;
  padding: 10px 0 2px;
  font-size: 18px;
}

.total-row strong {
  color: #f04438;
}

.submit-button {
  margin-top: 6px;
  padding: 13px;
}

.empty-cart,
.empty-state {
  padding: 40px 20px;
  border: 1px dashed #d9dee8;
  border-radius: 14px;
  color: #98a2b3;
  text-align: center;
}

.empty-cart span {
  font-size: 35px;
}

.empty-cart p {
  margin: 8px 0 0;
}

@media (max-width: 1000px) {
  .content-grid {
    grid-template-columns: 1fr;
  }

  .order-panel {
    position: static;
  }
}

@media (max-width: 680px) {
  .order-page {
    width: min(100% - 24px, 1400px);
    padding-top: 18px;
  }

  .order-hero {
    align-items: stretch;
    flex-direction: column;
    padding: 24px;
  }

  .cart-summary {
    text-align: left;
  }

  .dish-grid {
    grid-template-columns: 1fr;
  }
}
</style>
