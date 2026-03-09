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
    <span>天气数据获取失败</span>
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

// 取出天气平均值
const getTemperature = (min, max) => {
  try {
    // 计算平均值并四舍五入
    const average = (Number(min) + Number(max)) / 2;
    return Math.round(average);
  } catch (error) {
    console.error("计算温度出现错误：", error);
    return "NaN";
  }
};

// 使用浏览器定位获取天气
const getLocationByBrowser = () => {
  loading.value = true;
  
  if (!navigator.geolocation) {
    console.log("您的浏览器不支持地理定位功能");
    getLocationByIP();
    return;
  }
  
  navigator.geolocation.getCurrentPosition(
    async (position) => {
      try {
        const { latitude, longitude } = position.coords;
        console.log("获取到位置:", latitude, longitude);
        
        // 使用逆地理编码获取城市信息
        // 这里需要您实现或使用现有的逆地理编码API
        // 例如: const result = await getReverseGeocode(mainKey, `${longitude},${latitude}`);
        
        // 模拟逆地理编码结果
        const result = {
          regeocode: {
            addressComponent: {
              city: "上海市", // 这里应该是从API获取的实际城市
              adcode: "310100", // 这里应该是从API获取的实际adcode
            }
          },
          infocode: "10000"
        };
        
        if (result.infocode !== "10000") {
          throw new Error("逆地理编码失败");
        }
        
        weatherData.adCode = {
          city: result.regeocode.addressComponent.city,
          adcode: result.regeocode.addressComponent.adcode,
        };
        
        // 获取天气信息
        const weatherResult = await getWeather(mainKey, weatherData.adCode.adcode);
        weatherData.weather = {
          weather: weatherResult.lives[0].weather,
          temperature: weatherResult.lives[0].temperature,
          winddirection: weatherResult.lives[0].winddirection,
          windpower: weatherResult.lives[0].windpower,
        };
        
        loading.value = false;
      } catch (error) {
        console.error("浏览器定位获取天气失败:", error);
        // 失败后回退到IP定位
        getLocationByIP();
      }
    },
    (error) => {
      console.error("浏览器定位失败:", error);
      // 失败后回退到IP定位
      getLocationByIP();
    },
    {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 300000
    }
  );
};

// 使用IP定位获取天气
const getLocationByIP = async () => {
  loading.value = true;
  
  try {
    if (!mainKey) {
      console.log("未配置高德Key，使用备用天气接口");
      const result = await getOtherWeather();
      console.log(result);
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
      // 获取 Adcode
      const adCode = await getAdcode(mainKey);
      console.log(adCode);
      if (adCode.infocode !== "10000") {
        throw "地区查询失败";
      }
      weatherData.adCode = {
        city: adCode.city,
        adcode: adCode.adcode,
      };
      // 获取天气信息
      const result = await getWeather(mainKey, weatherData.adCode.adcode);
      weatherData.weather = {
        weather: result.lives[0].weather,
        temperature: result.lives[0].temperature,
        winddirection: result.lives[0].winddirection,
        windpower: result.lives[0].windpower,
      };
    }
    loading.value = false;
  } catch (error) {
    console.error("IP定位获取天气失败:" + error);
    onError("天气信息获取失败");
    loading.value = false;
  }
};

// 获取天气数据
const getWeatherData = () => {
  // 默认使用浏览器定位
  getLocationByBrowser();
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
  console.error(message);
};

onMounted(() => {
  // 调用获取天气
  getWeatherData();
});
</script>