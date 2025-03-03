<template>
  <div ref="outputDiv" class="code-output">
    <div>
      <div v-if="!kernel && !error" class="code-output__state-info">
        <bx-loading class="code-output__state-info__icon" assistive-text="Connecting to the server" type="small" />
        {{ $translate('Connecting to the server') }}
      </div>
      <div v-if="error !== ''" class="code-output__state-info">
        <p>{{ $translate('Failed to execute. Please refresh the page.') }}</p>
        {{ error }}
      </div>
    </div>
    <div ref="outputDiv" />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue-demi'
import { OutputArea, IOutputShellFuture, createOutputArea } from './OutputRenderer'
import { requestBinderKernel, IKernelConnection } from './KernelManager'
import 'carbon-web-components/es/components/loading/loading'

export default defineComponent({
  name: 'CodeOutput',
  data () {
    return {
      kernelPromise: undefined as Promise<IKernelConnection> | undefined,
      kernel: undefined as IKernelConnection | undefined,
      outputArea: undefined as OutputArea | undefined,
      error: ''
    }
  },
  mounted () {
    this.$emit('loadingKernel')
    const outputDivRef = (this.$refs.outputDiv as HTMLDivElement)

    this.kernelPromise = requestBinderKernel()

    this.kernelPromise.then((kernel: IKernelConnection) => {
      this.kernel = kernel
      this.$emit('kernelReady')
      this.outputArea = createOutputArea(outputDivRef)
    }, (error: Error) => {
      this.error = error.message
    })
  },
  methods: {
    requestExecute (code: string) {
      this.error = ''
      this.kernelPromise!.then((kernel: IKernelConnection) => {
        this.$emit('running')
        try {
          const requestFuture = kernel.requestExecute({ code })
          this.setOutputFuture(requestFuture)
          requestFuture.done.then(() => this.$emit('finished'))
        } catch (error: any) {
          this.error = error as string
          this.outputArea!.model.clear()
        }
      })
    },
    setOutputFuture (future : IOutputShellFuture) {
      this.outputArea!.future = future
    }
  }
})
</script>

<style lang="scss" scoped>
@import 'carbon-components/scss/globals/scss/spacing';
@import '~/../scss/variables/colors.scss';

.code-output {
  &__state-info {
    padding: $spacing-07;
    display: flex;
    align-items: center;
    gap: $spacing-05;

    &__icon {
      display: block;
      width: 1.5rem;
      height: 1.5rem;
      --cds-interactive-04: #{$border-color-tertiary};
    }
  }
}
</style>
