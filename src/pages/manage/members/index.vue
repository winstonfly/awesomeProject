<script setup lang="ts">
import { ref } from 'vue'
import { add, listUser, deleteUser } from '@/api/user'
import { listWs } from '@/api/workspace'
import { useTable, useAction } from '@/composables/useApi'
import { useConfirm } from '@/composables/useConfirm'
import { Users, Plus, RefreshCw, Settings, Trash2 } from 'lucide-vue-next'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import {
  Sheet, SheetContent, SheetHeader, SheetTitle, SheetDescription,
} from '@/components/ui/sheet'

definePage({
  meta: { title: '用户管理', description: '管理平台成员与 RBAC 权限。' },
})

const { confirm } = useConfirm()

// ── API 数据流 ────────────────────────────────────────────────────
const { rows: members, total, loading, params, refresh } = useTable(listUser)
const { rows: workspaces } = useTable(listWs)

const { loading: addLoading, execute: runAdd } = useAction(add, {
  successMsg: '成员添加成功',
  onSuccess: () => {
    sheetOpen.value = false
    refresh()
  },
})

// ── 状态控制 ──────────────────────────────────────────────────────
const sheetOpen = ref(false)
const sheetType = ref<'invite' | 'config'>('invite')
const selectedMember = ref<any>(null)

const form = ref({
  email: '',
  username: '',
  password: '',
  role: 'viewer',
  namespace: '',
  provider: 'local' as 'local' | 'dex',
})

function openInvite() {
  sheetType.value = 'invite'
  form.value = { email: '', username: '', password: '', role: 'viewer', namespace: '', provider: 'local' }
  sheetOpen.value = true
}

function openConfig(m: any) {
  selectedMember.value = JSON.parse(JSON.stringify(m))
  sheetType.value = 'config'
  sheetOpen.value = true
}

async function handleDelete(m: any) {
  const ok = await confirm({
    title: '确认移除成员？',
    message: `确认从团队中移除 ${m.name}？此操作将同步撤销其 RBAC 绑定。`,
    type: 'danger',
    confirmText: '确认移除',
  })
  if (ok) {
    await deleteUser(m.id)
    refresh()
  }
}

async function handleSave() {
  await runAdd(form.value)
}

// ── 样式辅助 ──────────────────────────────────────────────────────
const roleStyle: Record<string, string> = {
  admin:  'bg-primary/10 text-primary ring-1 ring-primary/20',
  editor: 'bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 ring-1 ring-emerald-500/20',
  viewer: 'bg-muted text-muted-foreground ring-1 ring-border',
}

const avatarColors = [
  'bg-blue-500', 'bg-violet-500', 'bg-emerald-500',
  'bg-orange-500', 'bg-rose-500', 'bg-cyan-500', 'bg-indigo-500',
]
function avatarColor(name: string) {
  let h = 0
  for (const c of name) h = (h * 31 + c.charCodeAt(0)) & 0xff
  return avatarColors[h % avatarColors.length]
}
function firstChar(name: string) {
  return name?.trim().charAt(0).toUpperCase() ?? '?'
}

// 命名空间 badge 颜色（按名字哈希）
function nsBadgeStyle(name: string) {
  const hues = [210, 160, 260, 40, 0, 190, 230]
  let h = 0
  for (const c of name) h = (h * 31 + c.charCodeAt(0)) & 0xff
  const hue = hues[h % hues.length]
  return {
    backgroundColor: `hsla(${hue}, 70%, 50%, 0.12)`,
    color: `hsla(${hue}, 80%, 60%, 1)`,
    outline: `1px solid hsla(${hue}, 70%, 50%, 0.2)`,
  }
}

// 分页
const totalPages = () => Math.ceil(total.value / params.pageSize)
</script>

<template>
  <div class="max-w-7xl mx-auto p-4 lg:p-8 space-y-6">

    <!-- ── Header ─────────────────────────────────────────────────── -->
    <header class="flex items-center justify-between pb-6 border-b border-border">
      <div class="flex items-center gap-4">
        <div class="p-3 bg-primary rounded-2xl text-primary-foreground shadow-lg shadow-primary/20">
          <Users class="size-6" />
        </div>
        <div>
          <h1 class="text-2xl font-black tracking-tighter uppercase">用户管理</h1>
          <p class="text-xs font-bold text-muted-foreground/50 uppercase tracking-widest">RBAC Control Plane</p>
        </div>
      </div>
      <div class="flex items-center gap-2">
        <Button variant="outline" size="sm" class="gap-1.5" :disabled="loading" @click="refresh">
          <RefreshCw class="size-3.5" :class="loading ? 'animate-spin' : ''" />
          刷新
        </Button>
        <Button size="sm" class="gap-1.5" @click="openInvite">
          <Plus class="size-3.5" /> 添加成员
        </Button>
      </div>
    </header>

    <!-- ── Table ──────────────────────────────────────────────────── -->
    <div class="bg-card border border-border rounded-2xl shadow-sm overflow-hidden">
      <div class="overflow-x-auto">
        <table class="w-full text-sm">
          <thead>
            <tr class="border-b border-border bg-muted/20">
              <th class="text-left px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">Identity</th>
              <th class="text-left px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50 hidden md:table-cell">Namespace Bindings</th>
              <th class="text-left px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">System Role</th>
              <th class="text-right px-6 py-3.5 text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">Actions</th>
            </tr>
          </thead>

          <tbody class="divide-y divide-border">
            <!-- Empty state -->
            <tr v-if="!loading && members.length === 0">
              <td colspan="4" class="py-20 text-center">
                <div class="flex flex-col items-center opacity-25">
                  <Users class="size-10 mb-3" />
                  <p class="text-sm font-black uppercase tracking-widest">No Members Found</p>
                </div>
              </td>
            </tr>

            <!-- Loading skeleton -->
            <tr v-else-if="loading && members.length === 0" v-for="i in 4" :key="i">
              <td colspan="4" class="px-6 py-4">
                <div class="h-4 bg-muted/50 rounded animate-pulse" />
              </td>
            </tr>

            <!-- Member rows -->
            <tr
              v-else
              v-for="m in members" :key="m.id"
              class="hover:bg-muted/30 transition-colors group"
            >
              <!-- Identity -->
              <td class="px-6 py-4">
                <div class="flex items-center gap-3">
                  <div
                    class="size-9 rounded-lg flex items-center justify-center text-white text-xs font-black shrink-0"
                    :class="avatarColor(m.name)"
                  >
                    {{ firstChar(m.name) }}
                  </div>
                  <div>
                    <p class="text-sm font-bold leading-none">{{ m.name }}</p>
                    <p class="text-[11px] font-mono text-muted-foreground/50 mt-0.5">{{ m.email }}</p>
                  </div>
                </div>
              </td>

              <!-- Namespace bindings -->
              <td class="px-6 py-4 hidden md:table-cell">
                <div class="flex flex-wrap gap-1.5">
                  <span
                    v-for="b in m.bindings" :key="b.ns"
                    :style="nsBadgeStyle(b.ns)"
                    class="px-2 py-0.5 rounded-md text-[10px] font-bold"
                  >
                    {{ b.ns }}
                  </span>
                  <span v-if="!m.bindings?.length" class="text-[11px] italic text-muted-foreground/30">Unassigned</span>
                </div>
              </td>

              <!-- Role -->
              <td class="px-6 py-4">
                <span
                  class="px-2.5 py-1 text-[10px] font-black rounded-lg uppercase tracking-wider"
                  :class="roleStyle[m.role] ?? roleStyle.viewer"
                >
                  {{ m.role ?? 'viewer' }}
                </span>
              </td>

              <!-- Actions -->
              <td class="px-6 py-4">
                <div class="flex items-center justify-end gap-1.5 opacity-0 group-hover:opacity-100 transition-opacity">
                  <Button
                    variant="ghost" size="sm"
                    class="h-7 text-xs text-primary hover:text-primary hover:bg-primary/10 gap-1"
                    @click="openConfig(m)"
                  >
                    <Settings class="size-3.5" /> 授权
                  </Button>
                  <Button
                    variant="ghost" size="sm"
                    class="h-7 w-7 p-0 text-muted-foreground/40 hover:text-destructive hover:bg-destructive/10"
                    @click="handleDelete(m)"
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
        <span class="text-xs text-muted-foreground/50">共 {{ total }} 位成员</span>
        <div class="flex items-center gap-1">
          <Button
            variant="outline" size="sm" class="h-7 text-xs px-2.5"
            :disabled="params.page <= 1"
            @click="params.page--"
          >
            上一页
          </Button>
          <span class="text-xs text-muted-foreground px-2">{{ params.page }} / {{ totalPages() || 1 }}</span>
          <Button
            variant="outline" size="sm" class="h-7 text-xs px-2.5"
            :disabled="params.page >= totalPages()"
            @click="params.page++"
          >
            下一页
          </Button>
        </div>
      </div>
    </div>

  </div>

  <!-- ── Sheet ──────────────────────────────────────────────────── -->
  <Sheet v-model:open="sheetOpen">
    <SheetContent class="w-[420px] sm:max-w-[420px] overflow-y-auto">

      <!-- Invite -->
      <template v-if="sheetType === 'invite'">
        <SheetHeader>
          <SheetTitle>添加成员</SheetTitle>
          <SheetDescription>向平台添加新用户并分配初始权限</SheetDescription>
        </SheetHeader>

        <div class="mt-6 space-y-5">
          <!-- Provider toggle -->
          <div class="flex bg-muted/50 rounded-xl p-1 border border-border">
            <button
              class="flex-1 py-2 rounded-lg text-xs font-black uppercase tracking-widest transition-all"
              :class="form.provider === 'local' ? 'bg-background text-primary shadow-sm' : 'text-muted-foreground'"
              @click="form.provider = 'local'"
            >
              Local
            </button>
            <button
              class="flex-1 py-2 rounded-lg text-xs font-black uppercase tracking-widest transition-all"
              :class="form.provider === 'dex' ? 'bg-background text-primary shadow-sm' : 'text-muted-foreground'"
              @click="form.provider = 'dex'"
            >
              OIDC / Dex
            </button>
          </div>

          <div class="space-y-1.5">
            <label class="text-xs font-medium text-foreground/70">用户名 / Email</label>
            <Input v-model="form.username" placeholder="ops@example.com" />
          </div>

          <div v-if="form.provider === 'local'" class="space-y-1.5">
            <label class="text-xs font-medium text-foreground/70">初始密码</label>
            <Input v-model="form.password" type="password" />
          </div>

          <div class="grid grid-cols-2 gap-3">
            <div class="space-y-1.5">
              <label class="text-xs font-medium text-foreground/70">系统角色</label>
              <select
                v-model="form.role"
                class="w-full h-9 rounded-md border border-input bg-background px-3 text-sm focus-visible:outline-none focus-visible:ring-[3px] focus-visible:ring-ring/50 transition-[color,box-shadow]"
              >
                <option value="admin">ADMIN</option>
                <option value="editor">EDITOR</option>
                <option value="viewer">VIEWER</option>
              </select>
            </div>
            <div class="space-y-1.5">
              <label class="text-xs font-medium text-foreground/70">初始空间</label>
              <select
                v-model="form.namespace"
                class="w-full h-9 rounded-md border border-input bg-background px-3 text-sm focus-visible:outline-none focus-visible:ring-[3px] focus-visible:ring-ring/50 transition-[color,box-shadow]"
              >
                <option value="">不指定</option>
                <option v-for="ws in workspaces" :key="ws.id" :value="ws.id">{{ ws.displayName }}</option>
              </select>
            </div>
          </div>

          <div class="flex gap-2 pt-2">
            <Button variant="outline" class="flex-1" @click="sheetOpen = false">取消</Button>
            <Button class="flex-1" :disabled="addLoading" @click="handleSave">
              <RefreshCw v-if="addLoading" class="size-3.5 animate-spin mr-2" />
              Provision Identity
            </Button>
          </div>
        </div>
      </template>

      <!-- Config -->
      <template v-else-if="selectedMember">
        <SheetHeader>
          <SheetTitle>RBAC 配置</SheetTitle>
          <SheetDescription>管理成员的命名空间绑定与权限</SheetDescription>
        </SheetHeader>

        <div class="mt-6 space-y-5">
          <!-- Member card -->
          <div class="flex items-center gap-4 p-4 bg-primary/5 rounded-xl border border-primary/10">
            <div
              class="size-12 rounded-xl flex items-center justify-center text-white text-base font-black shrink-0"
              :class="avatarColor(selectedMember.name)"
            >
              {{ firstChar(selectedMember.name) }}
            </div>
            <div>
              <p class="font-black text-sm">{{ selectedMember.name }}</p>
              <p class="text-xs font-mono text-muted-foreground/60">{{ selectedMember.email }}</p>
            </div>
          </div>

          <!-- Binding list -->
          <div>
            <div class="flex items-center justify-between mb-3">
              <p class="text-[10px] font-black uppercase tracking-widest text-muted-foreground/50">Namespace Bindings</p>
            </div>
            <div class="space-y-2">
              <div
                v-for="(b, i) in selectedMember.bindings" :key="i"
                class="p-3 rounded-xl border border-border bg-muted/20"
              >
                <div class="flex items-center justify-between">
                  <span class="text-xs font-bold uppercase italic">{{ b.ns }}</span>
                </div>
                <div class="grid grid-cols-2 gap-3 mt-2">
                  <div>
                    <p class="text-[10px] font-black text-muted-foreground/40 uppercase mb-1">RBAC Role</p>
                    <select class="w-full h-7 rounded-md border border-input bg-background px-2 text-xs focus-visible:outline-none">
                      <option>Admin</option>
                      <option>Editor</option>
                      <option>Viewer</option>
                    </select>
                  </div>
                  <div>
                    <p class="text-[10px] font-black text-muted-foreground/40 uppercase mb-1">Quota</p>
                    <p class="text-xs font-mono text-muted-foreground/60 mt-1.5">{{ b.quota || 'Standard' }}</p>
                  </div>
                </div>
              </div>
              <p v-if="!selectedMember.bindings?.length" class="text-xs text-muted-foreground/40 italic py-2">暂无命名空间绑定</p>
            </div>
          </div>

          <div class="flex gap-2 pt-2">
            <Button variant="outline" class="flex-1" @click="sheetOpen = false">取消</Button>
            <Button class="flex-1" @click="sheetOpen = false">Sync RBAC</Button>
          </div>
        </div>
      </template>

    </SheetContent>
  </Sheet>
</template>
