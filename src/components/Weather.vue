<template>
  <div class="weather" v-if="weatherData.adCode.city && weatherData.weather.weather">
    <span>{{ weatherData.adCode.city }}&nbsp;</span>
    <span>{{ weatherData.weather.weather }}&nbsp;</span>
    <span>{{ weatherData.weather.temperature }}℃</span>
    <span class="sm-hidden">
      &nbsp;{{
        weatherData.weather.winddirection?.endsWith("风")
          ? weatherData.weather.winddirection
          : weatherData.weather.winddirection + "风"
      }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.weather.windpower }}</span>
  </div>
  <div class="weather" v-else>
    <span>天气数据加载中...</span>
  </div>
</template>

<script setup>
import { reactive, onMounted, h } from "vue";
import axios from "axios"; // 建议直接使用 axios 或修改你的 api 封装
import { Error } from "@icon-park/vue-next";
import { ElMessage } from "element-plus";

// 天气数据结构
const weatherData = reactive({
  adCode: {
    city: null,
  },
  weather: {
    weather: null,
    temperature: null,
    winddirection: null,
    windpower: null,
  },
});

// 获取天气数据
const getWeatherData = async () => {
  try {
    // 使用新接口：uapis.cn
    // 注意：如果不传 city 或 adcode，接口会自动根据请求 IP 定位
    const response = await axios.get("https://uapis.cn/api/v1/misc/weather");
    
    if (response.status !== 200) {
      throw new Error("接口请求失败");
    }

    const data = response.data;

    // 映射数据
    weatherData.adCode = {
      city: data.city || "未知地区",
    };
    
    weatherData.weather = {
      weather: data.weather,
      temperature: data.temperature,
      winddirection: data.wind_direction,
      windpower: data.wind_power,
    };

  } catch (error) {
    console.error("天气信息获取失败:", error);
    onError("天气信息获取失败，请稍后再试");
  }
};

// 报错信息处理
const onError = (message) => {
  if (typeof ElMessage !== 'undefined') {
    ElMessage({
      message,
      icon: h(Error, {
        theme: "filled",
        fill: "#efefef",
      }),
    });
  }
};

onMounted(() => {
  getWeatherData();
});
</script>
