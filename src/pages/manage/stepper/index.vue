<script setup lang="ts">
import { ref, reactive, computed } from 'vue'
import { Check, Loader2, Copy, ShieldCheck, ArrowRight, ArrowLeft, Network } from 'lucide-vue-next'

definePage({
  meta: {
    title: '快速接入',
    description: '全域边缘节点接入引导程序。',
  },
})

const currentStep = ref(1)
const isJoining = ref(false)
const copied = ref(false)

const form = reactive({
  workspaceName: '',
  token: 'wf_live_8s2k...92nz',
  nodeLabel: 'edge-node-01',
})

const steps = [
  { id: 1, title: '工作空间', description: '定义逻辑边界' },
  { id: 2, title: '接入凭证', description: '生成认证 Token' },
  { id: 3, title: '执行加入', description: '运行接入命令' },
  { id: 4, title: '节点配置', description: '定义节点标签' },
  { id: 5, title: '完成', description: '拓扑同步成功' },
]

const canProceed = computed(() => {
  if (currentStep.value === 1) return !!form.workspaceName.trim()
  return true
})

const isLastStep = computed(() => currentStep.value === steps.length)
const isDone = computed(() => currentStep.value > steps.length)

function prevStep() {
  if (currentStep.value > 1) currentStep.value--
}

function nextStep() {
  if (currentStep.value < steps.length) currentStep.value++
  else currentStep.value = steps.length + 1 // mark done
}

function handleJoin() {
  isJoining.value = true
  setTimeout(() => {
    isJoining.value = false
    nextStep()
  }, 2000)
}

function copyToken() {
  navigator.clipboard.writeText(form.token)
  copied.value = true
  setTimeout(() => (copied.value = false), 2000)
}

const joinCommand = computed(
  () => `curl -sSL https://get.wireflow.io | sudo bash -s -- join --token ${form.token}`
)

function copyCommand() {
  navigator.clipboard.writeText(joinCommand.value)
  copied.value = true
  setTimeout(() => (copied.value = false), 2000)
}
</script>

<template>
  <div class="flex flex-col items-center p-6 gap-8 animate-in fade-in duration-500">

    <!-- ── Success state ──────────────────────────────────────────── -->
    <template v-if="isDone">
      <div class="w-full max-w-2xl">
        <div class="bg-card border border-border rounded-2xl p-12 text-center shadow-sm space-y-6 animate-in zoom-in-95 duration-500">
          <div class="relative mx-auto size-20">
            <div class="absolute inset-0 rounded-full bg-emerald-500/10 animate-ping opacity-30" />
            <div class="relative size-20 rounded-full bg-emerald-500/10 border border-emerald-500/20 flex items-center justify-center">
              <Check class="size-9 text-emerald-500 stroke-[2.5]" />
            </div>
          </div>
          <div class="space-y-2">
            <h2 class="text-3xl font-black tracking-tighter">接入成功</h2>
            <p class="text-muted-foreground text-sm max-w-sm mx-auto leading-relaxed">
              加密隧道建立完毕，节点 <code class="bg-muted px-1.5 py-0.5 rounded text-xs font-mono">{{ form.nodeLabel }}</code> 已注册至工作空间
              <code class="bg-muted px-1.5 py-0.5 rounded text-xs font-mono">{{ form.workspaceName }}</code>。
            </p>
          </div>
          <div class="pt-2 flex flex-col sm:flex-row gap-3 justify-center">
            <UiButton size="lg" class="gap-2">
              <Network class="size-4" /> 进入控制台
            </UiButton>
            <UiButton variant="outline" size="lg">下载接入报告</UiButton>
          </div>
        </div>
      </div>
    </template>

    <!-- ── Wizard ─────────────────────────────────────────────────── -->
    <template v-else>

      <!-- Step indicator -->
      <div class="w-full max-w-2xl">
        <div class="flex items-start">
          <template v-for="(step, i) in steps" :key="step.id">
            <!-- Node -->
            <div class="flex flex-col items-center gap-2 min-w-0">
              <div
                class="size-9 rounded-full flex items-center justify-center text-xs font-bold border-2 transition-all duration-300 shrink-0"
                :class="[
                  currentStep > step.id
                    ? 'bg-primary border-primary text-primary-foreground'
                    : currentStep === step.id
                      ? 'border-primary text-primary bg-primary/5 ring-4 ring-primary/10'
                      : 'border-border text-muted-foreground bg-background',
                ]"
              >
                <Check v-if="currentStep > step.id" class="size-4 stroke-[2.5]" />
                <span v-else>{{ step.id }}</span>
              </div>
              <div class="text-center px-1">
                <p
                  class="text-[11px] font-semibold leading-none transition-colors"
                  :class="currentStep === step.id ? 'text-foreground' : 'text-muted-foreground'"
                >
                  {{ step.title }}
                </p>
                <p class="text-[10px] text-muted-foreground/60 mt-0.5 hidden sm:block">{{ step.description }}</p>
              </div>
            </div>
            <!-- Connector -->
            <div
              v-if="i < steps.length - 1"
              class="flex-1 h-px mt-[17px] mx-1 transition-colors duration-500"
              :class="currentStep > step.id ? 'bg-primary' : 'bg-border'"
            />
          </template>
        </div>
      </div>

      <!-- Content card -->
      <div class="w-full max-w-2xl">
        <div class="bg-card border border-border rounded-2xl shadow-sm overflow-hidden">

          <!-- Card header strip -->
          <div class="border-b border-border/60 px-8 pt-8 pb-6">
            <div class="flex items-center gap-3">
              <span class="text-[10px] font-black tracking-[0.2em] uppercase text-muted-foreground/50 tabular-nums">
                STEP {{ String(currentStep).padStart(2, '0') }} / {{ String(steps.length).padStart(2, '0') }}
              </span>
              <div class="h-px flex-1 bg-border/40" />
            </div>
            <h2 class="mt-3 text-2xl font-black tracking-tight">
              <span v-if="currentStep === 1">命名您的工作空间</span>
              <span v-else-if="currentStep === 2">生成接入凭证</span>
              <span v-else-if="currentStep === 3">执行 Wireflow Join</span>
              <span v-else-if="currentStep === 4">配置节点元数据</span>
              <span v-else>验证节点状态</span>
            </h2>
            <p class="mt-1 text-sm text-muted-foreground">
              <span v-if="currentStep === 1">工作空间是节点和策略的逻辑边界，名称将用作路由标识符。</span>
              <span v-else-if="currentStep === 2">Token 是节点握手的唯一凭证，请妥善保管，不要泄露。</span>
              <span v-else-if="currentStep === 3">在目标机器终端执行以下命令，完成节点注册。</span>
              <span v-else-if="currentStep === 4">为节点设置标识符，便于在拓扑视图中识别和管理。</span>
              <span v-else>正在检测节点连接状态并同步拓扑信息。</span>
            </p>
          </div>

          <!-- Card body -->
          <div class="px-8 py-7">

            <!-- Step 1: Workspace -->
            <div v-if="currentStep === 1" class="space-y-4 animate-in slide-in-from-right-4 duration-300">
              <div class="space-y-1.5">
                <label class="text-xs font-semibold text-foreground/70 uppercase tracking-wider">工作空间名称</label>
                <UiInput
                  v-model="form.workspaceName"
                  placeholder="production-cluster-us"
                  class="h-11 font-mono text-sm bg-muted/20"
                  autofocus
                />
                <p class="text-xs text-muted-foreground/60">建议使用小写字母、数字和连字符，例如 <code class="font-mono">us-west-prod</code></p>
              </div>
            </div>

            <!-- Step 2: Token -->
            <div v-else-if="currentStep === 2" class="space-y-4 animate-in slide-in-from-right-4 duration-300">
              <div class="space-y-1.5">
                <label class="text-xs font-semibold text-foreground/70 uppercase tracking-wider">接入 Token</label>
                <div class="flex items-center gap-2 h-11 px-3 rounded-md border border-border bg-muted/20 font-mono text-sm">
                  <ShieldCheck class="size-4 text-emerald-500 shrink-0" />
                  <span class="flex-1 truncate text-foreground/60">{{ form.token }}</span>
                  <button
                    class="shrink-0 text-muted-foreground hover:text-foreground transition-colors"
                    @click="copyToken"
                  >
                    <Check v-if="copied" class="size-4 text-emerald-500" />
                    <Copy v-else class="size-4" />
                  </button>
                </div>
              </div>
              <div class="flex items-start gap-2.5 rounded-lg bg-amber-500/5 border border-amber-500/20 p-3.5">
                <ShieldCheck class="size-4 text-amber-500 mt-0.5 shrink-0" />
                <p class="text-xs text-amber-600/80 dark:text-amber-400/80 leading-relaxed">
                  此 Token 仅显示一次，请立即复制并保存至安全位置。
                </p>
              </div>
            </div>

            <!-- Step 3: Join command -->
            <div v-else-if="currentStep === 3" class="space-y-4 animate-in slide-in-from-right-4 duration-300">
              <div class="rounded-xl bg-zinc-950 border border-zinc-800 overflow-hidden shadow-lg">
                <!-- Terminal title bar -->
                <div class="flex items-center gap-1.5 px-4 py-3 border-b border-zinc-800/80">
                  <div class="size-3 rounded-full bg-zinc-700" />
                  <div class="size-3 rounded-full bg-zinc-700" />
                  <div class="size-3 rounded-full bg-zinc-700" />
                  <span class="ml-2 text-[11px] text-zinc-500 font-mono">bash</span>
                  <button
                    class="ml-auto flex items-center gap-1.5 text-[11px] text-zinc-500 hover:text-zinc-300 transition-colors"
                    @click="copyCommand"
                  >
                    <Check v-if="copied" class="size-3.5 text-emerald-400" />
                    <Copy v-else class="size-3.5" />
                    {{ copied ? 'Copied' : 'Copy' }}
                  </button>
                </div>
                <!-- Terminal content -->
                <div class="p-5">
                  <div class="flex gap-3 font-mono text-sm leading-relaxed">
                    <span class="text-zinc-500 select-none">$</span>
                    <code class="text-emerald-400/90 break-all">{{ joinCommand }}</code>
                  </div>
                </div>
              </div>
            </div>

            <!-- Step 4: Node label -->
            <div v-else-if="currentStep === 4" class="space-y-4 animate-in slide-in-from-right-4 duration-300">
              <div class="space-y-1.5">
                <label class="text-xs font-semibold text-foreground/70 uppercase tracking-wider">节点标识符</label>
                <UiInput
                  v-model="form.nodeLabel"
                  class="h-11 font-mono text-sm bg-muted/20"
                  placeholder="edge-node-01"
                />
                <p class="text-xs text-muted-foreground/60">此标识符将在拓扑视图和策略规则中引用。</p>
              </div>
            </div>

            <!-- Step 5: Verify -->
            <div v-else class="animate-in slide-in-from-right-4 duration-300">
              <div class="flex items-center gap-4 p-4 rounded-lg bg-muted/30 border border-border">
                <div class="size-10 rounded-full bg-muted flex items-center justify-center shrink-0">
                  <Loader2 v-if="isJoining" class="size-5 text-primary animate-spin" />
                  <Check v-else class="size-5 text-emerald-500" />
                </div>
                <div>
                  <p class="text-sm font-semibold">
                    {{ isJoining ? '正在检测节点连接...' : '准备就绪，点击完成接入' }}
                  </p>
                  <p class="text-xs text-muted-foreground mt-0.5">
                    {{ isJoining ? '通常需要 5–10 秒' : '系统将同步拓扑并完成注册' }}
                  </p>
                </div>
              </div>
            </div>

          </div>

          <!-- Card footer -->
          <div class="border-t border-border/60 px-8 py-5 flex items-center justify-between bg-muted/10">
            <UiButton
              variant="ghost"
              :disabled="currentStep === 1"
              class="gap-2 text-muted-foreground"
              @click="prevStep"
            >
              <ArrowLeft class="size-4" /> 上一步
            </UiButton>

            <div class="flex items-center gap-1.5">
              <div
                v-for="step in steps"
                :key="step.id"
                class="rounded-full transition-all duration-300"
                :class="
                  currentStep === step.id
                    ? 'w-5 h-1.5 bg-primary'
                    : currentStep > step.id
                      ? 'size-1.5 bg-primary/40'
                      : 'size-1.5 bg-border'
                "
              />
            </div>

            <!-- Step 3 has its own action button -->
            <UiButton
              v-if="currentStep === 3"
              :disabled="isJoining"
              class="gap-2"
              @click="handleJoin"
            >
              <Loader2 v-if="isJoining" class="size-4 animate-spin" />
              {{ isJoining ? '检测中...' : '检测并继续' }}
              <ArrowRight v-if="!isJoining" class="size-4" />
            </UiButton>

            <UiButton
              v-else-if="isLastStep"
              :disabled="!canProceed"
              class="gap-2"
              @click="nextStep"
            >
              完成接入 <Check class="size-4" />
            </UiButton>

            <UiButton
              v-else
              :disabled="!canProceed"
              class="gap-2"
              @click="nextStep"
            >
              继续 <ArrowRight class="size-4" />
            </UiButton>
          </div>

        </div>
      </div>

    </template>
  </div>
</template>
