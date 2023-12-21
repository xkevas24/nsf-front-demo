<template>
  <q-page :class="{ wave: color_mode, qzbg: !color_mode}">
    <div class="q-pa-md row items-start q-gutter-md" style="position: absolute;left: 35%;transform: translateX(-35%);">
      <q-card bordered style="width: 934px;">
        <q-card-section>
          <div class="text-h6">提供端服务状态检测 <span v-show="color_mode">- <q-icon name="colorize"></q-icon>流量染色模式：color={{ color_mark }}</span></div>
        </q-card-section>
        <q-separator inset />

        <q-card-section>
          <q-list bordered separator style="width: 900px;">
            <q-item clickable v-ripple>
              <q-tooltip class="bg-white text-black">实例IP：{{ product_info_ip }}，实例颜色：{{ product_info_color }}</q-tooltip>
              <q-item-section avatar>
                <q-icon name="apps" :class="product_info_app_color" />
              </q-item-section>
              <q-item-section>服务：product-info({{ product_info_ip[0] }})，版本：{{ product_info_version }}</q-item-section>
              <q-item-section>功能：{{ product_info_notes }}</q-item-section>
              <q-item-section side>
                <q-circular-progress
                  indeterminate
                  rounded
                  size="24px"
                  color="blue"
                  class="q-ma-md"
                  v-show="!product_info_status"
                />
                <q-icon name="check" color="green" v-show="product_info_status" />
              </q-item-section>
            </q-item>

            <q-item clickable v-ripple>
              <q-tooltip class="bg-white text-black">实例IP：{{ product_detail_ip }}，实例颜色：{{ product_detail_color }}</q-tooltip>
              <q-item-section avatar>
                <q-icon name="apps" :class="product_detail_app_color" /> <!--:color=""-->
              </q-item-section>
              <q-item-section>服务：product-detail({{ product_detail_ip[0] }})，版本：{{ product_detail_version }}</q-item-section>
              <q-item-section>功能：{{ product_detail_notes }}</q-item-section>
              <q-item-section side>
                <q-circular-progress
                  indeterminate
                  rounded
                  size="24px"
                  color="blue"
                  class="q-ma-md"
                  v-show="!product_detail_status"
                />
                <q-icon name="check" color="green" v-show="product_detail_status" />
              </q-item-section>
            </q-item>


          </q-list>
        </q-card-section>
      </q-card>

      <q-card bordered style="width: 934px; opacity: 0.8;">
        <q-card-section>
          <div class="row items-center no-wrap">
            <div class="col">
              <div class="text-h6">服务调用链示意图</div>
            </div>

            <div class="col-auto">
              <q-btn color="grey-7" :class="{ rotate_down: moc_visible, rotate_up: !moc_visible }" round flat icon="keyboard_arrow_up" @click="moc_showhide">
              </q-btn>
            </div>
          </div>
        </q-card-section>
        <q-card-section style="margin-top: -30px;">
          <q-toggle v-model="moc_visible" label="查看示意图" class="q-mb-md" v-show="false" />

          <q-slide-transition>
            <div v-show="moc_visible">
              <iframe :src="moc_url" frameborder="0" scrolling="no" style="width: 900px; height: 500px;"></iframe>
            </div>
          </q-slide-transition>


          
        </q-card-section>
      </q-card>

      <q-card bordered style="width: 934px;">
        <q-card-section>
          <div class="text-h6">product-info版本分流  
            <q-btn outline text-color="blue" icon="play_arrow" label="开始模拟" size="sm" style="margin-left: 30px;" @click="gate_open" v-show="!gate_status"></q-btn> 
            <q-btn outline text-color="red" icon="stop" label="停止模拟" size="sm" style="margin-left: 30px;" @click="gate_close" v-show="gate_status"></q-btn> 
          </div>
        </q-card-section>
        <q-separator inset />
        <q-card-section>
          <q-linear-progress size="25px" :value="product_info_v1" color="blue">
            <div class="absolute-full flex flex-center">
              <q-badge color="white" text-color="blue" :label="product_info_v1_label" />
            </div>
          </q-linear-progress>
        </q-card-section>
      </q-card>

    </div>
    <div class="q-pa-md row items-start q-gutter-md" style="position: absolute;left: 80%;transform: translateX(-50%);">
      <div style="background-image: url(ip14.png);background-size: cover; width: 390px; height: 792px;">
        <div class="q-pa-md q-gutter-md" v-show="chain_ok" style="padding-top: 90px; padding-left: 45px; height: 700px;">
          <q-scroll-area
            :thumb-style="thumbStyle"
            :bar-style="barStyle"
            style="height: 650px; max-width: 320px; z-index: 1;"
          >
          <div v-for="(item, index) in products" :key="index" style="margin-top: 20px;">
            <q-card class="my-card" style="width: 300px;">
              <q-img :src="item.img">
                <div class="absolute-bottom text-h6">
                  {{ item.title }}   年化{{ item.basic }}
                </div>
              </q-img>
              <q-card-section>
                {{ item.notice }}
              </q-card-section>
              <q-separator />
              <q-card-section>
                <div v-html="details[index].detail">

                </div>
                
              </q-card-section>
            </q-card>
          </div>

          </q-scroll-area>
        </div>
      </div>
    </div>
    <div v-show="chain_ok" style="position: absolute;left: 87%;transform: translateX(-50%);">
      <q-btn push round color="white" text-color="primary" icon="refresh" style="z-index: 999 !important; margin-top: 90px;" @click="get_product" />
    </div>
  </q-page>
</template>
<style>
  .rotate_down {
    transition: transform 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
    transform: rotate(360deg);
  }
  .rotate_up {
    transition: transform 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
    transform: rotate(180deg);
  }
</style>
<script>
import { defineComponent, onMounted, ref } from 'vue'
import { api } from 'boot/axios'
import { useQuasar } from 'quasar'


export default defineComponent({
  name: 'IndexPage',
  setup: () => {
    const $q = useQuasar()
    let iu = ref('poc-api:8080/api?entry=product-info:8080'); //8089
    let du = ref('poc-api:8080/api?entry=product-detail:8080'); //8089
    const searchParams = new URLSearchParams(window.location.search)
    const iu_user = searchParams.get('product_info')
    const du_user = searchParams.get('product_detail')
    if (iu_user !== null) {
      console.log(iu_user)
      iu.value = iu_user
    }

    if (du_user !== null) {
      console.log(du_user)
      du.value = du_user
    }

    let color_mode = ref(false)
    let color_mark = ref('')
    const color_mode_user = searchParams.get('color')
    if (color_mode_user !== null) {
      color_mode.value = true
      color_mark.value = color_mode_user
      console.log('已开启流量染色模式')
      $q.notify({
            icon: "colorize",
            message: `流量染色模式已启用，染色标识：${color_mark.value}`, 
            color:"purple", 
            position:"top-right"
      })
      iu.value = iu.value.replace("?entry=", `?color_mark=${color_mark.value}&entry=`)
      du.value = du.value.replace("?entry=", `?color_mark=${color_mark.value}&entry=`)
    }


    
    const product_info_uri = api.create({
        baseURL: `http://${iu.value}`,
        retry: 10,
        retryDelay: 1000
      })
    const product_detail_uri = api.create({
      baseURL: `http://${du.value}`,
      retry: 10,
      retryDelay: 1000
    })

    let moc_url = ref("");
    let product_info_version = ref("待检测")
    let product_info_status = ref(false)
    let product_info_notes = ref("")
    let product_info_ip = ref("")
    let product_info_color = ref("")
    let product_info_app_color = ref("")
    let product_info_v1 = ref(0)
    let pi_request_v1 = 0
    let pi_request_v2 = 0
    let product_info_v1_label = ref("V1.0.0 - 0%, V2.0.0 - 0%")

    let product_detail_version = ref("待检测")
    let product_detail_status = ref(false)
    let product_detail_notes = ref("")
    let product_detail_ip = ref("")
    let product_detail_color = ref("")
    let product_detail_app_color = ref("")
    
    let info_interval;
    let detail_interval;

    let chain_ok = ref(false)
    
    const product_info_ping = (notice=true) => {
      console.log('Ping product_info...');
      product_info_uri.get('/info/get_version')
      .then(res => {
        if (res.status == 200) {
          if (res.data.status == "429") {
            $q.notify({
              icon: "cancel",
              message: `/info/get_version 接口被限流`, 
              color:"red", 
              position:"top-right"
            })
          }

          product_info_version.value = res.data.data.version
          product_info_notes.value = res.data.data.notes
          product_info_status.value = true
          product_info_ip.value = res.data.data.ips
          clearInterval(info_interval)
          if (notice) {
            $q.notify({
              icon: "check_circle",
              message: `product-info:${product_info_version.value}已上线`, 
              color:"green", 
              position:"top-right"
            })
          }
          chain_ok.value = true
          moc_url.value = "https://modao.cc/flow/gWZFGCLds1mrd5f2Uyj9Uk"

          if (res.data.data.color_mark != null && res.data.data.color_mark != undefined && res.data.data.color_mark != "" && res.data.data.color_mark != [""]) {
            console.log('product-info 是染色实例')
            product_info_app_color.value = "blue-light"
            product_info_color.value = res.data.data.color_mark[0]
          } else {
            product_info_app_color.value = ""
            product_info_color.value = "无色"
          }

        }
      })
    }

    const product_detail_ping = (notice=true) => {
      console.log('Ping product_detail...');
      product_detail_uri.get('/detail/get_version')
      .then(res => {
        if (res.status == 200) {
          product_detail_version.value = res.data.data.version
          product_detail_notes.value = res.data.data.notes
          product_detail_status.value = true
          product_detail_ip.value = res.data.data.ips
          clearInterval(detail_interval)
          if (notice) {
            $q.notify({
              icon: "check_circle",
              message: `product-detail:${product_detail_version.value}已上线`, 
              color:"green", 
              position:"top-right"
            })
          }
          
          if (res.data.data.color_mark != null && res.data.data.color_mark != undefined && res.data.data.color_mark != "" && res.data.data.color_mark != [""]) {
            console.log('product-detail 是染色实例')
            product_detail_app_color.value = "blue-light"
            product_detail_color.value = res.data.data.color_mark
          } else {
            product_info_app_color.value = ""
            product_info_color.value = "无色"
          }
          if (notice) {
            get_product()
          }
        }
      })
    }
    setTimeout(() => {
      info_interval = setInterval(() => {
        product_info_ping()
      }, 1000)
    }, 2000)

    setTimeout(() => {
      detail_interval = setInterval(() => {
          product_detail_ping()
      }, 1000)
    }, 3000)
          

    let products = ref([])
    const get_product = () => {
      product_info_uri.get("/info/get_product")
      .then( res => {
        if (res.data.status == "429") {
            $q.notify({
              icon: "cancel",
              message: `/info/get_product 接口被限流`, 
              color:"red", 
              position:"top-right"
            })
          }
        products.value = res.data.data
        console.log(products.value)
        setTimeout(() => {
            get_detail()
          }, 1000)
      })

      // 刷新一下服务状态检测
      product_info_ping(false)
      product_detail_ping(false)
    }

    let details = ref([{"detail": "加载中"}, {"detail": "加载中"}, {"detail": "加载中"}, {"detail": "加载中"}])
    const get_detail = () => {
      product_detail_uri.get("/detail/get_product_detail")
      .then( res => {
        if (res.data == "local_rate_limited") {
          $q.notify({
            icon: "cancel",
            message: `product-detail:接口 /detail/get_product_detail 请求失败: ${err}`, 
            color:"red", 
            position:"top-right"
          })
          if (res.data.status == "429") {
            $q.notify({
              icon: "cancel",
              message: `/detail/get_product_detail 接口被限流`, 
              color:"red", 
              position:"top-right"
            })
          }
          details.value = [{"detail": "网络开了个小差"}, {"detail": "网络开了个小差"}, {"detail": "网络开了个小差"}, {"detail": "网络开了个小差"}]
        } else {
          details.value = res.data.data
          console.log(details.value)
        }
        
      })
      .catch( err => {
        $q.notify({
            icon: "cancel",
            message: `product-detail:接口 /detail/get_product_detail 请求失败: ${err}`, 
            color:"red", 
            position:"top-right"
          })
        details.value = [{"detail": "网络开了个小差"}, {"detail": "网络开了个小差"}, {"detail": "网络开了个小差"}, {"detail": "网络开了个小差"}]
      })
    }

    /* onMounted(product_info_ping);
    onMounted(product_detail_ping); */


    // 模拟请求
    let gate_status = ref(false)
    let gate_open_interval = null;
    const gate_open_func = () => {
      product_info_uri.get('/info/get_version')
      .then(res => {
        if (res.status == 200) {
          if (res.data.status == "429") {
            $q.notify({
              icon: "cancel",
              message: `/info/get_version 接口被限流`, 
              color:"red", 
              position:"top-right"
            })
          }
          product_info_version.value = res.data.data.version
          if (res.data.data.version == "V1.0.0") {
            pi_request_v1++
            $q.notify({
              message: `V1.0.0`, 
              color:"white", 
              textColor: "blue",
              position:"top-right"
            })
          } else {
            pi_request_v2++
            $q.notify({
              message: `V2.0.0`, 
              color:"white", 
              textColor: "green",
              position:"top-right"
            })
          }
          if (pi_request_v2 !== 0) {
            product_info_v1.value = pi_request_v1/(pi_request_v1 + pi_request_v2)
            let v1_percent = Math.round(product_info_v1.value * 100, 2)
            let v2_percent = Math.round((1 - product_info_v1.value) * 100, 2)
            product_info_v1_label.value = `V1.0.0 - ${v1_percent}%, V2.0.0 - ${v2_percent}%`
          } else {
            product_info_v1.value = 1
            product_info_v1_label.value = `V1.0.0 - 100%, V2.0.0 - 0%`
          }
        }
      })
    }
    const gate_open = () => {
      gate_status.value = true
      gate_open_interval = setInterval(() => {
        gate_open_func()
      }, 500)
    }

    const gate_close = () => {
      gate_status.value = false
      clearInterval(gate_open_interval)
      gate_open_interval = null
    }

    return {
      product_info_ping,
      product_info_version,
      product_info_status,
      product_info_notes,
      product_info_ip,
      product_info_color,
      product_info_v1,
      product_info_v1_label,
      product_info_app_color,
      product_detail_version,
      product_detail_status,
      product_detail_notes,
      product_detail_ip,
      product_detail_color,
      product_detail_app_color,
      chain_ok,
      moc_url,
      get_product,
      products,
      details,
      color_mode,
      color_mark,
      gate_open,
      gate_close,
      gate_status
    }
  },
  data: () => {
    return {
      moc_visible: true,
      moc_arrow: "keyboard_arrow_up",
      thumbStyle: {
        right: '4px',
        borderRadius: '5px',
        backgroundColor: '#027be3',
        width: '5px',
        opacity: 0.75
      },

      barStyle: {
        right: '2px',
        borderRadius: '9px',
        backgroundColor: '#027be3',
        width: '9px',
        opacity: 0.2
      }
    }
  },
  methods: {
    moc_showhide() {
      this.moc_visible = !this.moc_visible
      /* if (this.moc_visible) {
        // 示意图可见，箭头朝上
        this.moc_arrow = "keyboard_arrow_up"
      } else {
        this.moc_arrow = "keyboard_arrow_down"
      } */
    }
  }
})
</script>
