<template>
    <div v-if="pipelineSetting" class="bkdevops-base-info-setting-tab">
        <bk-form>
            <bk-form-item label="流水线名称" :required="true">
                <vuex-input placeholder="请输入流水线名称" name="pipelineName" :value="pipelineSetting.pipelineName" v-validate.initial="&quot;required|max:40&quot;" max-length="40" :handle-change="handleBaseInfoChange" />
            </bk-form-item>

            <bk-form-item :required="false" label="分组" v-if="tagGroupList.length">
                <div class="tag-group-row">
                    <div class="group-col" v-for="(filter, index) in tagGroupList" :key="index">
                        <label class="group-title">{{filter.name}}</label>
                        <bk-select
                            :value="pipelineSetting.labels"
                            @selected="handleLabelSelect"
                            multiple>
                            <bk-option v-for="item in filter.labels" :key="item.id" :id="item.id" :name="item.name"
                            ></bk-option>
                        </bk-select>
                    </div>
                </div>
            </bk-form-item>
            <bk-form-item label="描述" :is-error="errors.has(&quot;desc&quot;)" :error-msg="errors.first(&quot;desc&quot;)">
                <vuex-textarea name="desc" :value="pipelineSetting.desc" placeholder="请输入100个字符以内的描述内容" v-validate.initial="&quot;max:100&quot;" :handle-change="handleBaseInfoChange" />
            </bk-form-item>
            <bk-form-item class="item-badge" label="徽章" v-if="routeName !== 'templateSetting'">
                <img class="image-url" :src="badgeImageUrl">
                <div v-for="copyUrl in urlList" :key="copyUrl.url">
                    <label>{{copyUrl.label}}</label>
                    <p class="badge-item">
                        <bk-input readonly :value="copyUrl.url" disabled />
                        <span class="bk-icon icon-clipboard copy-icon" :data-clipboard-text="copyUrl.url"></span>
                    </p>
                </div>
            </bk-form-item>
        </bk-form>
    </div>
</template>

<script>
    import VuexTextarea from '@/components/atomFormField/VuexTextarea/index.vue'
    import VuexInput from '@/components/atomFormField/VuexInput/index.vue'
    import { mapGetters } from 'vuex'
    import Clipboard from 'clipboard'

    export default {
        name: 'bkdevops-base-info-setting-tab',
        components: {
            VuexTextarea,
            VuexInput
        },
        props: {
            pipelineSetting: Object,
            handleBaseInfoChange: Function
        },
        computed: {
            ...mapGetters({
                'tagGroupList': 'pipelines/getTagGroupList'
            }),
            projectId () {
                return this.$route.params.projectId
            },
            pipelineId () {
                return this.$route.params.pipelineId
            },
            badgeImageUrl () {
                return `${BADGE_URL_PREFIX}/process/api/external/pipelines/projects/${this.projectId}/${this.pipelineId}/badge?X-DEVOPS-PROJECT-ID=${this.projectId}`
            },
            badgeMarkdownLink () {
                return `[![BK Pipelines Status](${BADGE_URL_PREFIX}/process/api/external/pipelines/projects/${this.projectId}/${this.pipelineId}/badge?X-DEVOPS-PROJECT-ID=${this.projectId})](${AJAX_URL_PIRFIX}/process/api-html/user/builds/projects/${this.projectId}/pipelines/${this.pipelineId}/latestFinished?X-DEVOPS-PROJECT-ID=${this.projectId})`
            },
            urlList () {
                return [{
                    label: '图片url',
                    url: this.badgeImageUrl
                }, {
                    label: 'markdown链接',
                    url: this.badgeMarkdownLink
                }]
            }
        },
        created () {
            this.clipboard = new Clipboard('.copy-icon').on('success', e => {
                this.$showTips({
                    theme: 'success',
                    message: '内容复制成功'
                })
            })
            this.requestGrouptLists()
        },
        beforeDestroy () {
            this.clipboard.destroy()
        },
        methods: {
            /** *
             * 获取标签及其分组
             */
            async requestGrouptLists () {
                try {
                    const res = await this.$store.dispatch('pipelines/requestGetGroupLists', {
                        projectId: this.projectId
                    })
                    this.$store.commit('pipelines/updateGroupLists', res)
                    // this.dataList = this.tagGroupList
                } catch (err) {
                    this.$showTips({
                        message: err.message || err,
                        theme: 'error'
                    })
                }
            },
            handleLabelSelect (value) {
                this.handleBaseInfoChange('labels', value)
            }
        }
    }
</script>

<style lang="scss">
    .bkdevops-base-info-setting-tab {
        .form-group {
            margin-bottom: 12px;
            > label {
                margin-bottom: 5px;
                display: block;
            }
            > p {
                display: flex;
                align-items: center;
                > span {
                    margin-left: 12px;
                }
            }
        }
        .tag-group-row {
            display: flex;
            justify-content: space-around;
            .group-col {
                flex: 1;
                margin-right:8px;
                .group-title {
                    font-size: 12px;
                    line-height: 34px;
                }
                &:last-child {
                    margin-right: 0;
                }
            }
        }
        .item-badge {
            font-size: 12px;
            .image-url {
                margin: 7px 0
            }
            label {
                margin: 8px 0 0 0;
                display: block;
            }
            .badge-item {
                display: flex;
                align-items: center;
                .bk-icon {
                    font-size: 14px;
                    margin-left: 10px;
                }
            }
        }
    }
</style>
