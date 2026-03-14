<template>
    <div>
        <v-row class="pt-3">
            <v-col class="text-center">
                <v-btn
                    text
                    color="primary"
                    small
                    :disabled="['printing', 'paused'].includes(printer_state)"
                    @click="clickUpdate">
                    <v-icon left>{{ mdiProgressUpload }}</v-icon>
                    {{ $t('Machine.UpdatePanel.UpdateAll') }}
                </v-btn>
            </v-col>
        </v-row>
        <update-hint-all
            :bool-show-dialog="boolShowDialog"
            @close-dialog="boolShowDialog = false"
            @update-all="updateAll" />
    </div>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import { mdiProgressUpload } from '@mdi/js'
import UpdateHintAll from '@/components/panels/Machine/UpdatePanel/UpdateHintAll.vue'
@Component({
    components: { UpdateHintAll },
})
export default class UpdatePanelEntryAll extends Mixins(BaseMixin) {
    mdiProgressUpload = mdiProgressUpload

    boolShowDialog = false

    get hideUpdateWarning() {
        return this.$store.state.gui.uiSettings.hideUpdateWarnings ?? false
    }

    clickUpdate() {
        if (this.hideUpdateWarning) {
            this.updateAll()
            return
        }

        this.boolShowDialog = true
    }

    get modules() {
        return this.$store.getters['server/updateManager/getUpdateManagerList'] ?? []
    }

    updateAll() {
        // 逐个更新模块，跳过系统更新
        this.modules.forEach((module: any) => {
            if (['klipper', 'moonraker'].includes(module.name)) {
                this.$socket.emit('machine.update.' + module.name, {})
            } else {
                this.$socket.emit('machine.update.client', { name: module.name })
            }
        })
    }
}
</script>

<style scoped></style>
