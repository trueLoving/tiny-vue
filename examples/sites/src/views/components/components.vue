<template>
  <div>
    <div v-if="templateModeState.isSaas" class="ti-pt20 ti-pl48 ti-mb-36">
      <span class="cmp-mode-title">文档类型： </span>
      <tiny-button-group :data="optionsList" v-model="templateModeState.mode"></tiny-button-group>
    </div>
    <div class="ti-f-r ti-pt48 ti-pl48 ti-pr48">
      <div class="ti-fi-1 ti-w0 ti-rel cmp-container">
        <!-- 一个组件的文档:  描述md + demos + apis -->
        <div class="markdown-body markdown-top-body" size="medium" v-html="cmpTopMd"></div>
        <version-tip v-if="currJson.metaData || currJson.versionTipOption" :meta-data="currJson.metaData"
          v-bind="currJson.versionTipOption">
        </version-tip>
        <template v-if="currJson?.demos?.length > 0">
          <div class="all-demos-container">
            <h2 class="ti-f30 ti-fw-normal !ti-mb20">{{ i18nByKey('yan-shi') }}</h2>
            <div v-if="apiModeState.demoMode === 'default'" class="ti-f-c ti-f-wrap">
              <template v-if="currJson.column === '2' && currJson.demos.length > 1">
                <div class="one-demo-col2">
                  <div>
                    <demo v-for="demo in evenDemo" :key="demo.name" :demo="demo" />
                  </div>
                  <div>
                    <demo v-for="demo in oddDemo" :key="demo.name" :demo="demo" />
                  </div>
                </div>
              </template>
              <template v-else>
                <demo v-for="demo in currJson.demos" :key="demo.name" :demo="demo" class="mb16" />
              </template>
            </div>
            <div v-else>
              <demo v-if="singleDemo" :key="singleDemo.name" :demo="singleDemo" />
            </div>
          </div>
        </template>
        <template v-if="currJson.apis?.length > 0">
          <div id="API">
            <h2 class="ti-f30 ti-fw-normal ti-mt28">API</h2>
            <!-- apis 是一个数组 {name,type,properties:[原table内容],events:[] ...........} -->
            <div class="mt20" v-for="oneGroup in currJson.apis" :key="oneGroup.name">
              <div class="ti-f-r ti-f-pos-start ti-fw-bold">
                <div :id="oneGroup.name" class="ti-f18">{{ oneGroup.name }}</div>
                <div class="ti-ml12 ti-b-a-primary ti-c-primary ti-px8 ti-py4">{{ oneGroup.type }}</div>
              </div>
              <div v-for="(oneApiArr, key) in oneGroup" :key="key">
                <template v-if="key !== 'name' && key !== 'type' && oneApiArr.length > 0">
                  <div class="ti-f18 ti-py28">{{ key }}</div>
                  <table class="api-table">
                    <thead>
                      <tr v-if="key.includes('slots')">
                        <th width="15%">{{ i18nByKey('name') }}</th>
                        <th width="85%">{{ i18nByKey('desc') }}</th>
                      </tr>
                      <tr v-else-if="key.includes('events') || key.includes('methods')">
                        <th width="15%">{{ i18nByKey('name') }}</th>
                        <th width="20%">{{ i18nByKey('propType') }}</th>
                        <th width="65%">{{ i18nByKey('desc') }}</th>
                      </tr>
                      <tr v-else>
                        <th width="15%">{{ i18nByKey('name') }}</th>
                        <th width="20%" v-if="hasKey(oneApiArr, 'type')">{{ i18nByKey('propType') }}</th>
                        <th width="20%" v-if="hasKey(oneApiArr, 'defaultValue')">{{ i18nByKey('defValue') }}</th>
                        <th width="45%">{{ i18nByKey('desc') }}</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="row in oneApiArr.sort((a, b) => a.name.localeCompare(b.name))" :key="row.name">
                        <td>
                          <a v-if="row.demoId" @click="jumpToDemo(row.demoId)">{{ row.name }}</a>
                          <span v-else>{{ row.name }}</span>
                          <version-tip v-if="row.metaData || row.versionTipOption" :meta-data="row.metaData"
                            v-bind="row.versionTipOption" render-type="tag" tip-subject="api">
                          </version-tip>
                        </td>
                        <td v-if="!key.includes('slots') && hasKey(oneApiArr, 'type')" @click="handleTypeClick">
                          <a v-if="row.typeAnchorName"
                            :href="`${row.typeAnchorName.indexOf('#') === -1 ? '#' : ''}${row.typeAnchorName}`"
                            v-html="row.type"></a>
                          <span v-else v-html="row.type"></span>
                        </td>
                        <td v-if="
                          !key.includes('slots') &&
                          !key.includes('events') &&
                          !key.includes('methods') &&
                          hasKey(oneApiArr, 'defaultValue')
                        ">
                          <span
                            v-html="typeof row.defaultValue === 'string' ? row.defaultValue || '--' : row.defaultValue"></span>
                        </td>
                        <td><span v-html="row.desc[langKey]"></span></td>
                      </tr>
                    </tbody>
                  </table>
                </template>
              </div>
            </div>
          </div>
        </template>
        <template v-if="currJson.types && currJson.types.length">
          <div class="ti-f18 ti-py28" id="types">types</div>
          <tiny-collapse v-model="activeNames">
            <div v-for="typeItem in currJson.types" :id="typeItem.name" :key="typeItem.name">
              <tiny-collapse-item :title="typeItem.name" :name="typeItem.name">
                <async-highlight :code="typeItem.code.trim()" types="ts"></async-highlight></tiny-collapse-item>
            </div>
          </tiny-collapse>
        </template>
        <h2 id="FAQ" v-if="cmpFAQMd" class="ti-f30 ti-fw-normal ti-mt28 ti-mb20">FAQ</h2>
        <div class="markdown-body" v-html="cmpFAQMd"></div>
        <div v-if="currJson.owner" class="ti-abs ti-right24 ti-top24" @click="copyText(currJson.owner)">
          {{ i18nByKey('doc-owner') }} : {{ currJson.owner }}
        </div>
      </div>

      <!-- 目录列表 -->
      <div class="cmp-page-anchor catalog ti-w128 ti-sticky ti-top32" v-if="anchorLinks.length > 0">
        <tiny-anchor :is-affix="true" :links="anchorLinks" :key="anchorRefreshKey" mask-class="custom-active-anchor"
          @link-click="handleAnchorClick">
        </tiny-anchor>
      </div>
    </div>
    <div id="footer"></div>
  </div>
</template>

<script lang="jsx">
import { defineComponent, reactive, computed, toRefs, watch, onMounted, ref } from 'vue'
import { marked } from 'marked'
import { Anchor, ButtonGroup } from '@opentiny/vue'
import debounce from '@opentiny/vue-renderless/common/deps/debounce'
import { i18nByKey, getWord, $clone, fetchDemosFile, useApiMode, useTemplateMode } from '@/tools'
import demo from '@/views/components/demo'
import { router } from '@/router.js'
import { Collapse, CollapseItem } from '@opentiny/vue'
import { faqMdConfig, getWebdocPath } from './cmpConfig'
import AsyncHighlight from './async-highlight.vue'
import VersionTip from './VersionTip.vue'

export default defineComponent({
  name: 'CmpPageVue',
  components: {
    Demo: demo,
    TinyAnchor: Anchor,
    TinyButtonGroup: ButtonGroup,
    TinyCollapse: Collapse,
    TinyCollapseItem: CollapseItem,
    AsyncHighlight,
    VersionTip
  },
  setup() {
    const anchorRefreshKey = ref(0)
    const state = reactive({
      webDocPath: computed(() => ''),
      langKey: getWord('zh-CN', 'en-US'),
      cmpId: '',
      currJson: { column: 1, demos: [], apis: [] },
      cmpTopMd: null,
      cmpFAQMd: null,
      evenDemo: computed(() => state.currJson.demos?.filter((d, i) => i % 2 === 0) || []),
      oddDemo: computed(() => state.currJson.demos?.filter((d, i) => i % 2 === 1) || []),
      anchorLinks: computed(() => {
        const links =
          state.currJson?.demos?.map((demo) => ({
            key: demo.demoId,
            title: demo.name[state.langKey],
            link: `#${demo.demoId}`
          })) || []
        if (state.currJson?.apis?.length) {
          links.push({ key: 'API', title: 'API', link: '#API' })
        }
        if (state.currJson?.types?.length) {
          links.push({ key: 'types', title: 'types', link: '#types' })
        }
        if (state.cmpFAQMd) {
          links.push({
            key: 'FAQ',
            title: 'FAQ',
            link: '#FAQ'
          })
        }
        return links
      }),
      // 单demo显示时
      singleDemo: null,
      activeNames: ''
    })

    const { apiModeState } = useApiMode()
    const { templateModeState, staticPath, optionsList } = useTemplateMode()
    const huiNewChart = ['chart-process']

    // 页面加载/点击api中的链接，根据hash滚动。
    const scrollByHash = (hash) => {
      setTimeout(() => {
        if (!hash) {
          document.getElementById('doc-layout').scrollTo({
            top: 0,
            left: 0
          })
        } else {
          let scrollTarget
          try {
            //  用户打开官网有时候会带一些特殊字符的hash，try catch一下防止js报错
            scrollTarget = document.querySelector(`#${hash}`)
          } catch (err) {
            // eslint-disable-next-line no-console
            console.log('querySelector has special character:', err)
          }
          if (scrollTarget) {
            scrollTarget.scrollIntoView()
            document.getElementById('doc-layout').scrollTo({
              top: scrollTarget.offsetTop,
              left: 0,
              behavior: 'smooth'
            })
          }
        }
      }, 600)
    }

    // 在singleDemo情况时，才需要滚动示例区域到顶
    const scrollToLayoutTop = () => {
      let hash = router.currentRoute.value.hash?.slice(1)
      if (hash !== 'API') {
        setTimeout(() => {
          document.getElementById('doc-layout').scrollTo({
            top: 0,
            left: 0,
            behavior: 'smooth'
          })
        }, 0)
      }
    }

    // 改变折叠筐的展开状态， 2参为TRUE则表示是通过点击type属性跳转的锚点。不用进行判断直接赋值
    const changeActiveNames = (hash, isType = false) => {
      const newVal = hash.replace('#', '')
      if (isType) {
        state.activeNames = newVal
        return
      }

      const isTypeHashChange = state.currJson.types.some((item) => item.name === newVal)
      if (isTypeHashChange) {
        state.activeNames = newVal
      }
    }

    // saas下切换mode和组价示例都会触发loadPage,需要防抖
    const loadPage = debounce(templateModeState.isSaas ? 100 : 0, false, () => {
      const lang = getWord('cn', 'en')
      state.cmpId = router.currentRoute.value.params.cmpId

      // 将请求合并起来，这样页面更新一次，页面刷新的时机就固定了
      const promiseArr = [
        fetchDemosFile(`${staticPath.value}/${getWebdocPath(state.cmpId)}/webdoc/${state.cmpId}.${lang}.md`),
        fetchDemosFile(`${staticPath.value}/${getWebdocPath(state.cmpId)}/webdoc/${state.cmpId}.js`),
        fetchDemosFile(`@demos/apis/${getWebdocPath(state.cmpId) === 'chart' ? state.cmpId : getWebdocPath(state.cmpId)}.js`)
      ]
      if (faqMdConfig[state.cmpId]) {
        promiseArr.push(
          fetchDemosFile(`${staticPath.value}/${getWebdocPath(state.cmpId)}/webdoc/${state.cmpId}.faq.${lang}.md`)
        )
      }

      Promise.all(promiseArr).then(([mdData, jsData, apiData, faqData]) => {
        // 1、加载顶部md
        state.cmpTopMd = marked(mdData)

        // 2、加载faq.md
        if (faqData) {
          state.cmpFAQMd = marked(faqData)
        }

        // 3、加载cmpId.js 文件
        // eslint-disable-next-line no-eval
        const json = eval('(' + jsData.slice(15) + ')')
        state.currJson = {
          ...json,
          demos: $clone(json.demos || []), // 克隆一下,避免保存上次的isOpen
          column: json.column || '1' // columns可能为空
        }

        if (apiData) {
          // eslint-disable-next-line no-eval
          const apiJson = eval('(' + apiData.slice(15) + ')')
          // pc、mobile、mobile-first三种模式
          const demoMode = templateModeState.isSaas ? templateModeState.mode : import.meta.env.VITE_APP_MODE
          const demoKey = demoMode === 'mobile-first' ? 'mfDemo' : `${demoMode}Demo`
          state.currJson.apis = apiJson.apis.map((item) => {
            Object.keys(item).forEach((key) => {
              const apiItem = item[key]
              if (Array.isArray(apiItem)) {
                item[key] = apiItem
                  .filter((i) => !i.mode || i.mode.includes(demoMode))
                  .map((filterItem) => ({ ...filterItem, demoId: filterItem[demoKey] }))
              }
            })
            return item
          })
          state.currJson.types = apiJson.types
        }

        let hash = router.currentRoute.value.hash?.slice(1)

        // 单demo处理，如果有hash,取hash的demo, 没有hash, 取第1项
        if (hash) {
          state.singleDemo = state.currJson.demos.find((d) => d.demoId === hash)
          if (!state.singleDemo) {
            state.singleDemo = state.currJson.demos[0]
          }

          if (state.currJson.types && state.currJson.types.length) {
            changeActiveNames(hash)
          }
        } else {
          state.singleDemo = state.currJson.demos[0]
        }

        // F5刷新加载时，跳到当前示例
        scrollByHash(hash)
      })
    })
    const fn = {
      copyText: (text) => {
        navigator.clipboard.writeText(text)
      },
      // 点击 api区域的 name列时
      jumpToDemo: (demoId) => {
        if (demoId.startsWith('chart') || demoId.startsWith('grid')) {
          router.push(demoId)
        } else {
          router.push(`#${demoId}`)

          if (apiModeState.demoMode === 'single') {
            state.singleDemo = state.currJson.demos.find((d) => d.demoId === demoId)
          }
        }
      },
      // 点击api 区域的type列
      handleTypeClick: (ev) => {
        const hash = ev.target.hash
        if (hash) {
          changeActiveNames(hash, true)
        }
      },
      // 目录列表上的点击
      handleAnchorClick: (e, data) => {
        if (apiModeState.demoMode === 'single' && data.link.startsWith('#')) {
          e.preventDefault()
          const hash = data.link.slice(1)
          const singleDemo = state.currJson.demos.find((d) => d.demoId === hash)

          // 单示例模式下如果没有匹配到锚点对应的示例，则这不加载示例直接跳转锚点id
          if (singleDemo) {
            state.singleDemo = singleDemo
            scrollToLayoutTop()
          }

          router.push(data.link)
        } else if (apiModeState.demoMode === 'default' && data.link.startsWith('#')) {
          // 多示例模式，自动会切到相应的位置。只需要记录singleDemo就好了
          const hash = data.link.slice(1)
          state.singleDemo = state.currJson.demos.find((d) => d.demoId === hash)
        }
      }
    }

    watch(
      () => router.currentRoute.value.params.cmpId,
      (cmpId) => {
        if (!cmpId) {
          state.currJson = {}
        } else {
          loadPage()
          // 每次切换组件都需要让锚点组件重新刷新
          anchorRefreshKey.value++
        }
      }
    )
    watch(
      () => apiModeState.demoMode,
      (value) => {
        if (value) {
          scrollToLayoutTop()
        }
      }
    )

    watch(
      () => templateModeState.mode,
      () => {
        loadPage()
      }
    )

    onMounted(() => {
      loadPage()
      const common = new window.TDCommon(['#footer'], {})
      common.renderFooter()
    })

    const hasKey = (apiArr, key) => !apiArr.every((item) => item[key] === undefined)

    return {
      ...toRefs(state),
      ...fn,
      i18nByKey,
      anchorRefreshKey,
      apiModeState,
      templateModeState,
      optionsList,
      hasKey
    }
  }
})
</script>

<style lang="less" scoped>
table.api-table {
  width: 100%;
  table-layout: fixed;
  border-collapse: collapse;
  border: 1px solid rgb(239, 239, 245);

  a {
    text-decoration: none;
    color: #5e7ce0;
    cursor: pointer;
    word-wrap: break-word;
  }

  tbody tr:hover {
    background-color: rgb(250, 250, 252);
  }

  th {
    background-color: rgb(250, 250, 252);
    border-bottom: 1px solid rgb(239, 239, 245);
    padding: 12px;
    text-align: left;
    font-size: 14px;
    line-height: 1.5;
    word-break: break-word;
  }

  td {
    font-size: 14px;
    border-bottom: 1px solid rgb(239, 239, 245);
    padding: 12px;
    line-height: 1.5;

    .version-tip {
      margin-left: 6px;
    }
  }
}

.cmp-mode-title {
  font-size: 18px;
  vertical-align: middle;
  font-weight: 600;
}

.catalog {
  height: calc(100vh - 150px);
  overflow: hidden;
}

.catalog:hover {
  overflow-y: auto;
}

.catalog::-webkit-scrollbar {
  width: 10px;
  background-color: #f5f5f5;
}

.catalog::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background-color: #c1c1c1;
}

.one-demo-col2 {
  display: grid;
  gap: 16px;
  grid-template-columns: minmax(0px, 1fr) minmax(0px, 1fr);
  align-items: flex-start;

  >div {
    display: grid;
    gap: 16px;
    grid-template-columns: 100%;
  }
}

.cmp-container {
  padding-right: 24px;

  p {
    font-size: 16px;
    line-height: 1.7em;
    margin: 12px 0;
  }
}

.cmp-page-anchor {
  :deep(.tiny-anchor__affix) {
    top: unset !important;
    overflow-y: auto;
    max-height: 80vh;
  }

  :deep(.tiny-anchor-link) {
    margin-bottom: 10px;
    max-width: 150px;
    font-size: 12px;

    a {
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
  }
}

@media (max-width: 1279px) {
  .catalog {
    display: none;
  }

  .cmp-container {
    padding-right: 0;
  }
}

@media (max-width: 767px) {
  .one-demo-col2 {
    grid-template-columns: 100%;
  }
}

.all-demos-container {
  margin-top: 24px;
}

.custom-block.tip {
  background-color: #f3f5f7;
  border-color: #42b983;
  border-radius: 0.3rem;
  padding: 0.5rem 1rem;
  border-left-width: 0.3rem;
  border-left-style: solid;
  margin: 1rem 0;
  font-size: 14px;
  color: #5e6d82;
  line-height: 2;

  .custom-block-title {
    font-weight: 600;
    margin-bottom: 0.5rem;
  }

  p {
    margin: 0;
    font-size: 14px;
  }

  ul {
    li {
      padding: 5px 0;
    }
  }
}
</style>
