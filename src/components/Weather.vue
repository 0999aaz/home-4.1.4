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
    <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取中...</span>
  </div>
</template>

<script setup>
import { getAdcode, getWeather, getOtherWeather } from "@/api";
import { Error } from "@icon-park/vue-next";
import { ref, reactive, onMounted, h } from 'vue';
import { ElMessage } from 'element-plus';

// 高德开发者 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;

// 天气数据
const weatherData = reactive({
  adCode: {
    city: null, // 城市
    adcode: null, // 城市编码
  },
  weather: {
    weather: null, // 天气现象
    temperature: null, // 实时气温
    winddirection: null, // 风向描述
    windpower: null, // 风力级别
  },
});

// 加载状态
const loading = ref(false);

// 取出天气平均值 (用于备用接口)
const getTemperature = (min, max) => {
  try {
    const average = (Number(min) + Number(max)) / 2;
    return Math.round(average);
  } catch (error) {
    console.error("计算温度出现错误：", error);
    return "NaN";
  }
};

// 获取天气与定位数据 (统一使用 IP 定位)
const getWeatherData = async () => {
  loading.value = true;
  
  try {
    if (!mainKey) {
      console.log("未配置高德Key，使用备用天气接口");
      const result = await getOtherWeather();
      const data = result.result;
      weatherData.adCode = {
        city: data.city.City || "未知地区",
      };
      weatherData.weather = {
        weather: data.condition.day_weather,
        temperature: getTemperature(data.condition.min_degree, data.condition.max_degree),
        winddirection: data.condition.day_wind_direction,
        windpower: data.condition.day_wind_power,
      };
    } else {
      // 1. 获取基于 IP 的定位 Adcode
      const adCodeResult = await getAdcode(mainKey);
      if (adCodeResult.infocode !== "10000") {
        throw new Error("地区查询失败");
      }

      // 修复直辖市逻辑（city 为空时取 province）
      const locationName = (adCodeResult.city && typeof adCodeResult.city === 'string') 
        ? adCodeResult.city 
        : adCodeResult.province;

      weatherData.adCode = {
        city: locationName,
        adcode: adCodeResult.adcode,
      };

      // 2. 获取天气信息
      const weatherResult = await getWeather(mainKey, weatherData.adCode.adcode);
      if (weatherResult.infocode !== "10000") {
        throw new Error("天气获取失败");
      }

      weatherData.weather = {
        weather: weatherResult.lives[0].weather,
        temperature: weatherResult.lives[0].temperature,
        winddirection: weatherResult.lives[0].winddirection,
        windpower: weatherResult.lives[0].windpower,
      };
    }
  } catch (error) {
    console.error("定位或获取天气失败:", error);
    onError("天气信息获取失败");
  } finally {
    loading.value = false;
  }
};

// 报错信息
const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
};

onMounted(() => {
  // 组件挂载时调用
  getWeatherData();
});
</script>
