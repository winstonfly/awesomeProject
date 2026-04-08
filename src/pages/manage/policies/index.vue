<script setup lang="ts">
import { onMounted } from 'vue'
import { Shield, Plus, RefreshCw, Pencil, Trash2, ArrowRightLeft, ArrowDown, ArrowUp, Eye } from 'lucide-vue-next'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import {
  Sheet, SheetContent, SheetHeader, SheetTitle, SheetDescription,
} from '@/components/ui/sheet'
import { toast } from 'vue-sonner'
import { usePolicyPageStore } from '@/stores/usePolicyPageStore'

definePage({
  meta: { title: '策略管理', description: '管理网络访问控制策略。' },
})

const store = usePolicyPageStore()
onMounted(() => store.actions.refresh())

// ── 样式辅助 ──────────────────────────────────────────────────────
const actionStyle: Record<string, string> = {
  Allow: 'bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 ring-1 ring-emerald-500/20',
  Deny:  'bg-rose-500/10 text-rose-600 dark:text-rose-400 ring-1 ring-rose-500/20',
}

const typeColors: Record<string, string> = {
  Ingress: 'bg-blue-500/10 text-blue-600 dark:text-blue-400 ring-1 ring-blue-500/20',
  Egress:  'bg-violet-500/10 text-violet-600 dark:text-violet-400 ring-1 ring-violet-500/20',
}

// 模板快速创建
const templates = [
  { key: 'isolate', label: '全隔离',    desc: 'Deny All' },
  { key: 'db',      label: '数据库保护', desc: 'DB Ingress' },
  { key: 'internet',label: '放通出口',   desc: 'Allow Egress' },
]

// 分页
const totalPages = () => Math.ceil(store.total / store.params.pageSize)
</script>

<template>
  <div class="max-w-7xl mx-auto p-4 lg:p-8 space-y-6">

    <!-- ── Header ─────────────────────────────────────────────────── -->
    <header class="flex items-center justify-between pb-6 border-b border-border">
      <div class="flex items-center gap-4">
        <div class="p-3 bg-primary rounded-2xl text-primary-foreground shadow-lg shadow-primary/20">
          <Shield class="size-6" />
        </div>
        <div>
          <h1 class="text-2xl font-black tracking-tighter uppercase">策略管理</h1>
          <p class="text-xs font-bold text-muted-foreground/50 uppercase tracking-widest">Network Access Policy</p>
        </div>
      </div>
      <div class="flex items-center gap-2">
        <Button variant="outline" size="sm" class="gap-1.5" :disabled="store.loading" @click="store.actions.refresh()">
          <RefreshCw class="size-3.5" :class="store.loading ? 'animate-spin' : ''" />
          刷新
        </Button>
        <Button size="sm" class="gap-1.5" @click="store.actions.openDrawer('create')">
          <Plus class="size-3.5" /> 新建策略
        </Button>
      </div>
    </header>

    <!-- ── Policy table ───────────────────────────────────────────── -->
    <div class="bg-card border border-border rounded-2xl shadow-sm overflow-hidden">
      <div class="overflow-x-auto">
        <table class="w-full text-sm">
          <thead>
            <tr class="border-b border-border bg-muted/20">
              <th class="text-left px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">策略名称</th>
              <th class="text-left px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50 hidden md:table-cell">目标选择器</th>
              <th class="text-left px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50 hidden lg:table-cell">策略类型</th>
              <th class="text-left px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">动作</th>
              <th class="text-right px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">Actions</th>
            </tr>
          </thead>

          <tbody class="divide-y divide-border">
            <!-- Empty -->
            <tr v-if="!store.loading && store.rows.length === 0">
              <td colspan="5" class="py-20 text-center">
                <div class="flex flex-col items-center opacity-25">
                  <Shield class="size-10 mb-3" />
                  <p class="text-sm font-black uppercase tracking-widest">No Policies Found</p>
                </div>
              </td>
            </tr>

            <!-- Loading -->
            <tr v-else-if="store.loading && store.rows.length === 0" v-for="i in 4" :key="i">
              <td colspan="5" class="px-6 py-4">
                <div class="h-4 bg-muted/50 rounded animate-pulse" />
              </td>
            </tr>

            <!-- Rows -->
            <tr
              v-else
              v-for="policy in store.rows" :key="policy.name"
              class="hover:bg-muted/30 transition-colors group"
            >
              <!-- Name -->
              <td class="px-6 py-4">
                <div class="flex items-center gap-3">
                  <div class="size-8 rounded-lg bg-primary/10 flex items-center justify-center shrink-0">
                    <Shield class="size-4 text-primary" />
                  </div>
                  <div>
                    <p class="text-sm font-bold leading-none">{{ policy.name }}</p>
                    <p v-if="policy.description" class="text-[11px] text-muted-foreground/50 mt-0.5">{{ policy.description }}</p>
                  </div>
                </div>
              </td>

              <!-- Target selector -->
              <td class="px-6 py-4 hidden md:table-cell">
                <div class="flex flex-wrap gap-1.5">
                  <span
                    v-for="(val, key) in policy.peerSelector?.matchLabels" :key="key"
                    class="px-2 py-0.5 rounded-md text-[10px] font-bold font-mono bg-muted/60 text-muted-foreground ring-1 ring-border"
                  >
                    {{ key }}={{ val }}
                  </span>
                </div>
              </td>

              <!-- Policy types -->
              <td class="px-6 py-4 hidden lg:table-cell">
                <div class="flex gap-1.5">
                  <span
                    v-for="t in policy.policyTypes" :key="t"
                    class="px-2 py-0.5 rounded-md text-[10px] font-bold flex items-center gap-1"
                    :class="typeColors[t] ?? 'bg-muted text-muted-foreground'"
                  >
                    <ArrowDown v-if="t === 'Ingress'" class="size-3" />
                    <ArrowUp v-else class="size-3" />
                    {{ t }}
                  </span>
                </div>
              </td>

              <!-- Action -->
              <td class="px-6 py-4">
                <span
                  class="px-2.5 py-1 text-[10px] font-black rounded-lg uppercase tracking-wider"
                  :class="actionStyle[policy.action] ?? actionStyle.Allow"
                >
                  {{ policy.action ?? 'Allow' }}
                </span>
              </td>

              <!-- Actions -->
              <td class="px-6 py-4">
                <div class="flex items-center justify-end gap-1.5 opacity-0 group-hover:opacity-100 transition-opacity">
                  <Button
                    variant="ghost" size="sm"
                    class="h-7 text-xs text-muted-foreground hover:text-foreground gap-1"
                    @click="store.actions.openDrawer('view', policy)"
                  >
                    <Eye class="size-3.5" /> 查看
                  </Button>
                  <Button
                    variant="ghost" size="sm"
                    class="h-7 text-xs text-primary hover:text-primary hover:bg-primary/10 gap-1"
                    @click="store.actions.openDrawer('edit', policy)"
                  >
                    <Pencil class="size-3.5" /> 编辑
                  </Button>
                  <Button
                    variant="ghost" size="sm"
                    class="h-7 w-7 p-0 text-muted-foreground/40 hover:text-destructive hover:bg-destructive/10"
                    @click="store.actions.handleDelete(policy, toast)"
                  >
                    <Trash2 class="size-3.5" />
                  </Button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pagination -->
      <div class="border-t border-border px-6 py-3 bg-muted/10 flex items-center justify-between">
        <span class="text-xs text-muted-foreground/50">共 {{ store.total }} 条策略</span>
        <div class="flex items-center gap-1">
          <Button
            variant="outline" size="sm" class="h-7 text-xs px-2.5"
            :disabled="store.params.page <= 1"
            @click="store.params.page--"
          >
            上一页
          </Button>
          <span class="text-xs text-muted-foreground px-2">{{ store.params.page }} / {{ totalPages() || 1 }}</span>
          <Button
            variant="outline" size="sm" class="h-7 text-xs px-2.5"
            :disabled="store.params.page >= totalPages()"
            @click="store.params.page++"
          >
            下一页
          </Button>
        </div>
      </div>
    </div>

  </div>

  <!-- ── Create / Edit / View Sheet ────────────────────────────── -->
  <Sheet :open="store.ui.isDrawerOpen" @update:open="v => { if (!v) store.ui.isDrawerOpen = false }">
    <SheetContent class="w-[540px] sm:max-w-[540px] overflow-y-auto">
      <SheetHeader class="mb-6">
        <SheetTitle>
          {{ store.ui.drawerType === 'create' ? '新建策略' : store.ui.drawerType === 'edit' ? '编辑策略' : '策略详情' }}
        </SheetTitle>
        <SheetDescription>
          {{ store.ui.drawerType === 'create' ? '定义一条 WireflowPolicy 网络访问控制规则' : '查看或修改策略配置' }}
        </SheetDescription>
      </SheetHeader>

      <div class="space-y-6">

        <!-- Quick templates (only for create) -->
        <div v-if="store.ui.drawerType === 'create'" class="space-y-2">
          <p class="text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">快速模板</p>
          <div class="grid grid-cols-3 gap-2">
            <button
              v-for="tpl in templates" :key="tpl.key"
              class="p-3 rounded-xl border border-border bg-muted/20 hover:border-primary/40 hover:bg-primary/5 transition-all text-left"
              @click="store.actions.applyTemplate(tpl.key)"
            >
              <p class="text-xs font-bold">{{ tpl.label }}</p>
              <p class="text-[10px] text-muted-foreground/60 mt-0.5">{{ tpl.desc }}</p>
            </button>
          </div>
        </div>

        <!-- Basic fields -->
        <div class="space-y-4">
          <div class="space-y-1.5">
            <label class="text-xs font-medium text-foreground/70">策略名称</label>
            <Input
              v-model="store.form.name"
              placeholder="例如：deny-all-egress"
              class="font-mono text-xs"
              :disabled="store.ui.drawerType === 'view'"
            />
          </div>

          <div class="space-y-1.5">
            <label class="text-xs font-medium text-foreground/70">目标节点选择器 <span class="font-mono text-[10px] text-muted-foreground">(key=value)</span></label>
            <Input
              v-model="store.form._targetLabel"
              placeholder="app=web"
              class="font-mono text-xs"
              :disabled="store.ui.drawerType === 'view'"
            />
          </div>

          <!-- Policy types -->
          <div class="space-y-1.5">
            <label class="text-xs font-medium text-foreground/70">策略方向</label>
            <div class="flex gap-2">
              <label
                v-for="t in ['Ingress', 'Egress']" :key="t"
                class="flex items-center gap-2 px-3 py-2 rounded-lg border cursor-pointer transition-all"
                :class="store.form.policyTypes.includes(t)
                  ? (t === 'Ingress' ? 'border-blue-500/40 bg-blue-500/5' : 'border-violet-500/40 bg-violet-500/5')
                  : 'border-border bg-muted/10 opacity-50'"
              >
                <input
                  type="checkbox"
                  :checked="store.form.policyTypes.includes(t)"
                  :disabled="store.ui.drawerType === 'view'"
                  class="sr-only"
                  @change="store.form.policyTypes.includes(t)
                    ? store.form.policyTypes.splice(store.form.policyTypes.indexOf(t), 1)
                    : store.form.policyTypes.push(t)"
                />
                <component
                  :is="t === 'Ingress' ? ArrowDown : ArrowUp"
                  class="size-3.5"
                  :class="t === 'Ingress' ? 'text-blue-500' : 'text-violet-500'"
                />
                <span class="text-xs font-bold">{{ t }}</span>
              </label>
            </div>
          </div>
        </div>

        <!-- Rules: Ingress -->
        <div v-if="store.form.policyTypes.includes('Ingress')" class="space-y-2">
          <div class="flex items-center justify-between">
            <p class="text-[10px] font-black uppercase tracking-widest text-muted-foreground/50 flex items-center gap-1.5">
              <ArrowDown class="size-3 text-blue-500" /> Ingress Rules
            </p>
            <Button
              v-if="store.ui.drawerType !== 'view'"
              variant="ghost" size="sm" class="h-6 text-[10px] text-primary font-black uppercase"
              @click="store.actions.addRule('ingress')"
            >
              + ADD
            </Button>
          </div>
          <div
            v-for="(rule, i) in store.form.ingress" :key="i"
            class="p-3 rounded-xl border border-border bg-muted/10 space-y-2"
          >
            <div class="grid grid-cols-2 gap-2">
              <div class="space-y-1">
                <p class="text-[10px] text-muted-foreground/50 font-semibold uppercase">来源选择器</p>
                <Input v-model="rule._rawLabel" placeholder="app=web" class="h-7 text-xs font-mono" :disabled="store.ui.drawerType === 'view'" />
              </div>
              <div class="space-y-1">
                <p class="text-[10px] text-muted-foreground/50 font-semibold uppercase">端口</p>
                <Input v-model="rule.ports[0].port" placeholder="80" class="h-7 text-xs font-mono" :disabled="store.ui.drawerType === 'view'" />
              </div>
            </div>
          </div>
          <p v-if="!store.form.ingress.length" class="text-xs text-muted-foreground/40 italic py-1">无规则 — 将拒绝所有入站流量</p>
        </div>

        <!-- Rules: Egress -->
        <div v-if="store.form.policyTypes.includes('Egress')" class="space-y-2">
          <div class="flex items-center justify-between">
            <p class="text-[10px] font-black uppercase tracking-widest text-muted-foreground/50 flex items-center gap-1.5">
              <ArrowUp class="size-3 text-violet-500" /> Egress Rules
            </p>
            <Button
              v-if="store.ui.drawerType !== 'view'"
              variant="ghost" size="sm" class="h-6 text-[10px] text-primary font-black uppercase"
              @click="store.actions.addRule('egress')"
            >
              + ADD
            </Button>
          </div>
          <div
            v-for="(rule, i) in store.form.egress" :key="i"
            class="p-3 rounded-xl border border-border bg-muted/10 space-y-2"
          >
            <div class="grid grid-cols-2 gap-2">
              <div class="space-y-1">
                <p class="text-[10px] text-muted-foreground/50 font-semibold uppercase">目标选择器</p>
                <Input v-model="rule._rawLabel" placeholder="app=db" class="h-7 text-xs font-mono" :disabled="store.ui.drawerType === 'view'" />
              </div>
              <div class="space-y-1">
                <p class="text-[10px] text-muted-foreground/50 font-semibold uppercase">端口</p>
                <Input v-model="rule.ports[0].port" placeholder="5432" class="h-7 text-xs font-mono" :disabled="store.ui.drawerType === 'view'" />
              </div>
            </div>
          </div>
          <p v-if="!store.form.egress.length" class="text-xs text-muted-foreground/40 italic py-1">无规则 — 将拒绝所有出站流量</p>
        </div>

        <!-- YAML Preview -->
        <div class="space-y-1.5">
          <p class="text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">YAML Preview</p>
          <pre class="bg-zinc-950 dark:bg-zinc-900 text-emerald-400 text-[11px] font-mono rounded-xl p-4 overflow-x-auto border border-zinc-800 leading-relaxed">{{ store.yamlPreview }}</pre>
        </div>

        <!-- Footer buttons -->
        <div v-if="store.ui.drawerType !== 'view'" class="flex gap-2 pt-2">
          <Button variant="outline" class="flex-1" @click="store.ui.isDrawerOpen = false">取消</Button>
          <Button
            class="flex-1"
            :disabled="store.loading"
            @click="store.actions.handleCreateOrUpdate(toast)"
          >
            <RefreshCw v-if="store.loading" class="size-3.5 animate-spin mr-2" />
            {{ store.ui.drawerType === 'create' ? '发布策略' : '保存更改' }}
          </Button>
        </div>
        <Button v-else variant="outline" class="w-full" @click="store.ui.isDrawerOpen = false">关闭</Button>

      </div>
    </SheetContent>
  </Sheet>
</template>
