<script setup lang="ts">
import { computed } from 'vue'

definePage({
  meta: {
    title: 'Dashboard',
    description: "Welcome back! Here's what's happening today.",
  },
})
import {
  TrendingUp, TrendingDown, DollarSign, Users, ShoppingCart,
  CreditCard, ArrowUpRight, ArrowDownRight, MoreHorizontal,
} from 'lucide-vue-next'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'

// ── Stats ────────────────────────────────────────────────────────────────────
const stats = [
  {
    title: 'Total Revenue',
    value: '$45,231.89',
    change: '+20.1%',
    trend: 'up' as const,
    description: 'from last month',
    icon: DollarSign,
    sparkline: [30, 45, 35, 50, 40, 65, 60, 75, 55, 80, 70, 85],
  },
  {
    title: 'Active Users',
    value: '+2,350',
    change: '+180.1%',
    trend: 'up' as const,
    description: 'from last month',
    icon: Users,
    sparkline: [20, 30, 25, 40, 35, 50, 45, 60, 55, 65, 60, 70],
  },
  {
    title: 'New Orders',
    value: '+12,234',
    change: '+19%',
    trend: 'up' as const,
    description: 'from last month',
    icon: ShoppingCart,
    sparkline: [40, 35, 50, 45, 60, 55, 65, 60, 75, 70, 80, 85],
  },
  {
    title: 'Conversion Rate',
    value: '3.24%',
    change: '-4.75%',
    trend: 'down' as const,
    description: 'since last month',
    icon: CreditCard,
    sparkline: [70, 65, 75, 60, 70, 55, 65, 50, 60, 45, 55, 40],
  },
]

// ── Chart helpers ────────────────────────────────────────────────────────────
function buildPath(data: number[], w: number, h: number, pad = 8) {
  const max = Math.max(...data)
  const min = Math.min(...data)
  const range = max - min || 1
  const xStep = (w - pad * 2) / (data.length - 1)
  const pts = data.map((v, i) => ({
    x: pad + i * xStep,
    y: h - pad - ((v - min) / range) * (h - pad * 2),
  }))
  const line = pts.map((p, i) => `${i === 0 ? 'M' : 'L'}${p.x.toFixed(1)},${p.y.toFixed(1)}`).join(' ')
  const area = `${line} L${pts.at(-1)!.x.toFixed(1)},${h - pad} L${pts[0].x.toFixed(1)},${h - pad} Z`
  return { line, area, pts }
}

function sparkline(data: number[]) {
  return buildPath(data, 80, 28, 2).line
}

// ── Revenue area chart ───────────────────────────────────────────────────────
const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
const revenueValues = [18000, 22000, 19000, 28000, 24000, 32000, 30000, 35000, 29000, 38000, 36000, 45000]
const prevRevenueValues = [12000, 16000, 14000, 20000, 18000, 25000, 22000, 28000, 24000, 30000, 28000, 35000]

const revenueChart = computed(() => buildPath(revenueValues, 520, 180, 16))
const prevRevenueChart = computed(() => buildPath(prevRevenueValues, 520, 180, 16))

// ── Bar chart (orders by day) ────────────────────────────────────────────────
const days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
const ordersData = [120, 180, 150, 220, 195, 260, 240]
const ordersMax = Math.max(...ordersData)

// ── Transactions ─────────────────────────────────────────────────────────────
const transactions = [
  { id: '#3210', name: 'Olivia Martin', email: 'olivia.martin@email.com', type: 'Sale', status: 'Approved', date: 'Jan 8, 2024', amount: '+$1,999.00', initials: 'OM', pos: true },
  { id: '#3209', name: 'Jackson Lee', email: 'jackson.lee@email.com', type: 'Refund', status: 'Declined', date: 'Jan 7, 2024', amount: '-$39.00', initials: 'JL', pos: false },
  { id: '#3208', name: 'Isabella Nguyen', email: 'isabella.nguyen@email.com', type: 'Sale', status: 'Approved', date: 'Jan 7, 2024', amount: '+$299.00', initials: 'IN', pos: true },
  { id: '#3207', name: 'William Kim', email: 'will@email.com', type: 'Sale', status: 'Approved', date: 'Jan 6, 2024', amount: '+$99.00', initials: 'WK', pos: true },
  { id: '#3206', name: 'Sofia Davis', email: 'sofia.davis@email.com', type: 'Sale', status: 'Pending', date: 'Jan 6, 2024', amount: '+$39.00', initials: 'SD', pos: true },
]

const statusStyle: Record<string, string> = {
  Approved: 'bg-emerald-100 text-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-400',
  Declined: 'bg-red-100 text-red-700 dark:bg-red-900/30 dark:text-red-400',
  Pending: 'bg-amber-100 text-amber-700 dark:bg-amber-900/30 dark:text-amber-400',
}

// ── Top Products ──────────────────────────────────────────────────────────────
const products = [
  { name: 'Glimmer Lamps', sales: 1204, revenue: '$29,890', growth: '+12.5%', trend: 'up' as const },
  { name: 'Aqua Filters', sales: 967, revenue: '$18,420', growth: '+8.2%', trend: 'up' as const },
  { name: 'Eco Planters', sales: 842, revenue: '$15,670', growth: '-3.4%', trend: 'down' as const },
  { name: 'Solar Torches', sales: 738, revenue: '$12,800', growth: '+5.1%', trend: 'up' as const },
  { name: 'Spark Plugs', sales: 621, revenue: '$9,340', growth: '-1.2%', trend: 'down' as const },
]
</script>

<template>
  <div class="flex flex-col gap-5 p-6">

    <!-- ── Stats Cards ─────────────────────────────────────────────────── -->
    <div class="grid gap-4 sm:grid-cols-2 xl:grid-cols-4">
      <div
        v-for="stat in stats"
        :key="stat.title"
        class="border-border bg-card rounded-xl border p-5"
      >
        <div class="flex items-start justify-between">
          <div class="flex flex-col gap-1">
            <span class="text-muted-foreground text-sm font-medium">{{ stat.title }}</span>
            <span class="text-2xl font-bold tracking-tight">{{ stat.value }}</span>
          </div>
          <div class="bg-muted rounded-lg p-2">
            <component :is="stat.icon" class="text-muted-foreground size-4" />
          </div>
        </div>

        <div class="mt-3 flex items-center justify-between">
          <div class="flex items-center gap-1 text-sm">
            <component
              :is="stat.trend === 'up' ? ArrowUpRight : ArrowDownRight"
              :class="stat.trend === 'up' ? 'text-emerald-600' : 'text-red-500'"
              class="size-4"
            />
            <span :class="stat.trend === 'up' ? 'text-emerald-600' : 'text-red-500'" class="font-semibold">
              {{ stat.change }}
            </span>
            <span class="text-muted-foreground">{{ stat.description }}</span>
          </div>
        </div>

        <!-- Sparkline -->
        <svg class="mt-3 w-full" viewBox="0 0 80 28" preserveAspectRatio="none" style="height:28px">
          <path
            :d="sparkline(stat.sparkline)"
            fill="none"
            :stroke="stat.trend === 'up' ? '#10b981' : '#ef4444'"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </div>
    </div>

    <!-- ── Charts Row ──────────────────────────────────────────────────── -->
    <div class="grid gap-4 lg:grid-cols-3">

      <!-- Revenue Area Chart -->
      <div class="border-border bg-card rounded-xl border p-5 lg:col-span-2">
        <div class="mb-4 flex items-start justify-between">
          <div>
            <h3 class="font-semibold">Revenue Overview</h3>
            <p class="text-muted-foreground text-sm">Monthly revenue for 2024</p>
          </div>
          <div class="flex items-center gap-4 text-xs">
            <div class="flex items-center gap-1.5">
              <span class="inline-block size-2.5 rounded-full bg-primary"></span>
              <span class="text-muted-foreground">This year</span>
            </div>
            <div class="flex items-center gap-1.5">
              <span class="inline-block size-2.5 rounded-full bg-blue-300"></span>
              <span class="text-muted-foreground">Last year</span>
            </div>
          </div>
        </div>

        <svg viewBox="0 0 520 180" class="w-full" style="height:180px" preserveAspectRatio="xMidYMid meet">
          <defs>
            <linearGradient id="revGrad" x1="0" y1="0" x2="0" y2="1">
              <stop offset="0%" stop-color="var(--primary)" stop-opacity="0.3" />
              <stop offset="100%" stop-color="var(--primary)" stop-opacity="0" />
            </linearGradient>
            <linearGradient id="prevGrad" x1="0" y1="0" x2="0" y2="1">
              <stop offset="0%" stop-color="#93c5fd" stop-opacity="0.25" />
              <stop offset="100%" stop-color="#93c5fd" stop-opacity="0" />
            </linearGradient>
          </defs>

          <!-- Grid lines -->
          <line v-for="i in 4" :key="i" :y1="i * 40" :y2="i * 40" x1="16" x2="504"
            stroke="currentColor" stroke-opacity="0.08" stroke-width="1" />

          <!-- Previous year area -->
          <path :d="prevRevenueChart.area" fill="url(#prevGrad)" />
          <path :d="prevRevenueChart.line" fill="none" stroke="#93c5fd" stroke-width="1.5"
            stroke-linejoin="round" stroke-linecap="round" stroke-dasharray="4 3" />

          <!-- Current year area -->
          <path :d="revenueChart.area" fill="url(#revGrad)" />
          <path :d="revenueChart.line" fill="none" stroke="var(--primary)" stroke-width="2"
            stroke-linejoin="round" stroke-linecap="round" />

          <!-- Dots on current year -->
          <circle
            v-for="(pt, i) in revenueChart.pts"
            :key="i"
            :cx="pt.x"
            :cy="pt.y"
            r="3"
            fill="var(--primary)"
            class="opacity-0 transition-opacity hover:opacity-100"
          />
        </svg>

        <!-- X labels -->
        <div class="mt-1 flex justify-between px-4 text-xs text-muted-foreground">
          <span v-for="m in months" :key="m">{{ m }}</span>
        </div>
      </div>

      <!-- Orders Bar Chart -->
      <div class="border-border bg-card rounded-xl border p-5">
        <div class="mb-4">
          <h3 class="font-semibold">Orders This Week</h3>
          <p class="text-muted-foreground text-sm">Daily order volume</p>
        </div>

        <div class="flex h-40 items-end gap-2">
          <div
            v-for="(val, i) in ordersData"
            :key="days[i]"
            class="flex flex-1 flex-col items-center gap-1"
          >
            <span class="text-muted-foreground text-[10px] font-medium">{{ val }}</span>
            <div
              class="bg-primary/80 hover:bg-primary w-full rounded-t transition-colors"
              :style="{ height: `${(val / ordersMax) * 120}px` }"
            />
          </div>
        </div>

        <div class="mt-2 flex justify-between text-xs text-muted-foreground">
          <span v-for="d in days" :key="d">{{ d }}</span>
        </div>

        <div class="border-border mt-4 border-t pt-4">
          <div class="flex items-center justify-between">
            <div>
              <p class="text-sm font-semibold">1,365</p>
              <p class="text-muted-foreground text-xs">Total this week</p>
            </div>
            <div class="flex items-center gap-1 text-sm text-emerald-600">
              <TrendingUp class="size-4" />
              <span class="font-semibold">+12.5%</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ── Bottom Row ──────────────────────────────────────────────────── -->
    <div class="grid gap-4 lg:grid-cols-3">

      <!-- Recent Transactions -->
      <div class="border-border bg-card rounded-xl border lg:col-span-2">
        <div class="flex items-center justify-between border-b border-border p-5">
          <div>
            <h3 class="font-semibold">Recent Transactions</h3>
            <p class="text-muted-foreground text-sm">5 transactions this week</p>
          </div>
          <button class="text-muted-foreground hover:text-foreground rounded-md p-1 transition-colors">
            <MoreHorizontal class="size-4" />
          </button>
        </div>

        <div class="overflow-x-auto">
          <table class="w-full text-sm">
            <thead>
              <tr class="border-border border-b">
                <th class="text-muted-foreground px-5 py-3 text-left text-xs font-medium uppercase tracking-wider">Customer</th>
                <th class="text-muted-foreground px-3 py-3 text-left text-xs font-medium uppercase tracking-wider">Type</th>
                <th class="text-muted-foreground px-3 py-3 text-left text-xs font-medium uppercase tracking-wider">Status</th>
                <th class="text-muted-foreground px-3 py-3 text-left text-xs font-medium uppercase tracking-wider">Date</th>
                <th class="text-muted-foreground px-5 py-3 text-right text-xs font-medium uppercase tracking-wider">Amount</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="tx in transactions"
                :key="tx.id"
                class="border-border hover:bg-muted/30 border-b transition-colors last:border-0"
              >
                <td class="px-5 py-3">
                  <div class="flex items-center gap-3">
                    <Avatar class="size-8">
                      <AvatarFallback class="text-xs">{{ tx.initials }}</AvatarFallback>
                    </Avatar>
                    <div>
                      <p class="font-medium">{{ tx.name }}</p>
                      <p class="text-muted-foreground text-xs">{{ tx.email }}</p>
                    </div>
                  </div>
                </td>
                <td class="px-3 py-3 text-muted-foreground">{{ tx.type }}</td>
                <td class="px-3 py-3">
                  <span :class="statusStyle[tx.status]" class="rounded-full px-2.5 py-0.5 text-xs font-medium">
                    {{ tx.status }}
                  </span>
                </td>
                <td class="px-3 py-3 text-muted-foreground text-xs">{{ tx.date }}</td>
                <td class="px-5 py-3 text-right font-medium" :class="tx.pos ? 'text-emerald-600' : 'text-red-500'">
                  {{ tx.amount }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Top Products -->
      <div class="border-border bg-card rounded-xl border p-5">
        <div class="mb-4 flex items-center justify-between">
          <div>
            <h3 class="font-semibold">Top Products</h3>
            <p class="text-muted-foreground text-sm">By revenue this month</p>
          </div>
          <button class="text-muted-foreground hover:text-foreground rounded-md p-1 transition-colors">
            <MoreHorizontal class="size-4" />
          </button>
        </div>

        <div class="flex flex-col gap-4">
          <div
            v-for="(product, i) in products"
            :key="product.name"
            class="flex items-center gap-3"
          >
            <div class="bg-muted text-muted-foreground flex size-8 shrink-0 items-center justify-center rounded-lg text-xs font-bold">
              {{ i + 1 }}
            </div>
            <div class="min-w-0 flex-1">
              <p class="truncate text-sm font-medium">{{ product.name }}</p>
              <p class="text-muted-foreground text-xs">{{ product.sales }} sales</p>
            </div>
            <div class="text-right">
              <p class="text-sm font-semibold">{{ product.revenue }}</p>
              <div class="flex items-center justify-end gap-0.5 text-xs"
                :class="product.trend === 'up' ? 'text-emerald-600' : 'text-red-500'">
                <component :is="product.trend === 'up' ? TrendingUp : TrendingDown" class="size-3" />
                <span>{{ product.growth }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Donut-style summary -->
        <div class="border-border mt-4 border-t pt-4">
          <div class="mb-2 flex justify-between text-xs text-muted-foreground">
            <span>Total Revenue</span>
            <span class="font-medium text-foreground">$86,120</span>
          </div>
          <div class="bg-muted h-2 overflow-hidden rounded-full">
            <div class="h-full rounded-full bg-primary" style="width: 72%"></div>
          </div>
          <p class="mt-1 text-right text-xs text-muted-foreground">72% of monthly target</p>
        </div>
      </div>
    </div>

  </div>
</template>
