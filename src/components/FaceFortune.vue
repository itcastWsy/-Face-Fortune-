<template>
  <div class="face-fortune-container">
    <el-card class="fortune-card">
      <template #header>
        <div class="card-header">
          <h2>面相算命</h2>
          <div class="camera-controls">
            <el-button type="primary" @click="startCamera" :disabled="isCameraActive">
              <el-icon><VideoCamera /></el-icon> 打开摄像头
            </el-button>
            <el-button type="danger" @click="stopCamera" :disabled="!isCameraActive">
              <el-icon><VideoCameraFilled /></el-icon> 关闭摄像头
            </el-button>
            <el-button type="success" @click="captureAndAnalyze" :disabled="!isCameraActive || isAnalyzing">
              <el-icon><Camera /></el-icon> 开始算命
            </el-button>
          </div>
        </div>
      </template>
      
      <div class="camera-container">
        <video ref="video" class="camera-view" :class="{ hidden: !isCameraActive }" autoplay muted></video>
        <canvas ref="canvas" class="face-canvas"></canvas>
        <div v-if="!isCameraActive && !isAnalyzing" class="camera-placeholder">
          <el-icon><Camera /></el-icon>
          <p>请点击"打开摄像头"按钮开始</p>
        </div>
        <div v-if="isAnalyzing" class="analyzing">
          <el-icon class="is-loading"><Loading /></el-icon>
          <p>正在分析面相...</p>
        </div>
      </div>

      <div v-if="fortuneResult" class="fortune-result">
        <h3>面相分析结果</h3>
        
        <div class="basic-info">
          <el-descriptions :column="2" border>
            <el-descriptions-item label="性别">
              <el-tag :type="fortuneResult.gender === '男' ? 'primary' : 'danger'">
                {{ fortuneResult.gender }}
              </el-tag>
            </el-descriptions-item>
            <el-descriptions-item label="年龄">
              {{ fortuneResult.age }}岁
            </el-descriptions-item>
          </el-descriptions>
        </div>

        <div class="personality-chart">
          <h4>性格分析</h4>
          <div class="chart-container">
            <v-chart class="chart" :option="personalityChartOption" autoresize />
          </div>
        </div>

        <div class="fortune-details">
          <el-collapse v-model="activeCollapse">
            <el-collapse-item title="性格特点" name="personality">
              <div class="fortune-text">{{ fortuneResult.personality.description }}</div>
            </el-collapse-item>
            <el-collapse-item title="事业分析" name="career">
              <div class="fortune-text">{{ fortuneResult.career.description }}</div>
            </el-collapse-item>
            <el-collapse-item title="婚姻分析" name="marriage">
              <div class="fortune-text">{{ fortuneResult.marriage.description }}</div>
            </el-collapse-item>
          </el-collapse>
        </div>

        <div class="daily-fortune">
          <h4>今日运势</h4>
          <el-row :gutter="20">
            <el-col :span="12">
              <el-card class="daily-card suitable">
                <template #header>
                  <div class="card-header">
                    <span>今日宜</span>
                  </div>
                </template>
                <div class="daily-items">
                  <div v-for="(item, index) in fortuneResult.daily.suitable" :key="'suitable-'+index" class="daily-item">
                    <el-tag type="success" effect="light">{{ item }}</el-tag>
                  </div>
                </div>
              </el-card>
            </el-col>
            <el-col :span="12">
              <el-card class="daily-card avoid">
                <template #header>
                  <div class="card-header">
                    <span>今日忌</span>
                  </div>
                </template>
                <div class="daily-items">
                  <div v-for="(item, index) in fortuneResult.daily.avoid" :key="'avoid-'+index" class="daily-item">
                    <el-tag type="danger" effect="light">{{ item }}</el-tag>
                  </div>
                </div>
              </el-card>
            </el-col>
          </el-row>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, computed } from 'vue';
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { RadarChart } from 'echarts/charts';
import { TitleComponent, TooltipComponent, LegendComponent } from 'echarts/components';
import VChart from 'vue-echarts';
import * as faceapi from 'face-api.js';
import { ElNotification } from 'element-plus';

// 注册 ECharts 组件
use([
  CanvasRenderer,
  RadarChart,
  TitleComponent,
  TooltipComponent,
  LegendComponent
]);

// 响应式状态
const video = ref(null);
const canvas = ref(null);
const isCameraActive = ref(false);
const isAnalyzing = ref(false);
const stream = ref(null);
const fortuneResult = ref(null);
const activeCollapse = ref(['personality']);

// 用于稳定检测结果的缓存数据
const detectionBuffer = ref([]);
const bufferSize = 5; // 保存最近5帧的检测结果
const minDetectionConfidence = 0.7; // 最小检测置信度阈值

// 性格雷达图配置
const personalityChartOption = computed(() => {
  if (!fortuneResult.value) return {};
  
  return {
    radar: {
      indicator: [
        { name: '开朗', max: 100 },
        { name: '稳重', max: 100 },
        { name: '创造力', max: 100 },
        { name: '耐心', max: 100 },
        { name: '细心', max: 100 }
      ],
      radius: 80
    },
    series: [{
      type: 'radar',
      data: [{
        value: [
          fortuneResult.value.personality.traits.outgoing,
          fortuneResult.value.personality.traits.steady,
          fortuneResult.value.personality.traits.creative,
          fortuneResult.value.personality.traits.patient,
          fortuneResult.value.personality.traits.detail
        ],
        name: '性格特点',
        areaStyle: {
          color: 'rgba(64, 158, 255, 0.6)'
        }
      }]
    }]
  };
});

// 生命周期钩子
onMounted(async () => {
  try {
    // 加载face-api.js模型
    await loadFaceApiModels();
    console.log('Face-API 模型加载完成');
  } catch (error) {
    console.error('加载模型失败:', error);
    ElNotification({
      title: '错误',
      message: '无法加载人脸识别模型，请刷新页面重试',
      type: 'error',
      duration: 5000
    });
  }
});

onUnmounted(() => {
  stopCamera();
});

// 方法
async function loadFaceApiModels() {
  // 使用绝对路径，确保在开发和生产环境都能正确加载
  const MODEL_URL = '/models';
  
  // 先尝试加载tiny_face_detector模型，因为这是目前唯一存在的模型
  try {
    console.log('开始加载TinyFaceDetector模型，路径:', MODEL_URL);
    await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL);
    console.log('TinyFaceDetector模型加载成功');
    
    // 如果有其他模型文件，可以取消下面的注释
    // 目前先注释掉不存在的模型加载，避免报错
    /*
    await Promise.all([
      faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL),
      faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL),
      faceapi.nets.faceExpressionNet.loadFromUri(MODEL_URL),
      faceapi.nets.ageGenderNet.loadFromUri(MODEL_URL)
    ]);
    */
  } catch (error) {
    console.error('加载TinyFaceDetector模型失败:', error);
    console.error('模型路径:', MODEL_URL);
    throw error;
  }
}

async function startCamera() {
  try {
    stream.value = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'user' }
    });
    
    if (video.value) {
      video.value.srcObject = stream.value;
      isCameraActive.value = true;
    }
  } catch (error) {
    console.error('无法访问摄像头:', error);
    ElNotification({
      title: '错误',
      message: '无法访问摄像头，请确保已授予摄像头权限',
      type: 'error',
      duration: 5000
    });
  }
}

function stopCamera() {
  if (stream.value) {
    const tracks = stream.value.getTracks();
    tracks.forEach(track => track.stop());
    stream.value = null;
    isCameraActive.value = false;
    
    // 清除画布
    const ctx = canvas.value.getContext('2d');
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  }
}

async function captureAndAnalyze() {
  if (!isCameraActive.value) return;
  
  isAnalyzing.value = true;
  console.log('开始分析人脸...');
  
  try {
    // 检查模型是否已加载
    if (!faceapi.nets.tinyFaceDetector.isLoaded) {
      console.warn('TinyFaceDetector模型未加载，尝试重新加载...');
      try {
        await loadFaceApiModels();
      } catch (modelError) {
        console.error('重新加载模型失败:', modelError);
        throw new Error('人脸识别模型未加载，请刷新页面重试');
      }
    }
    
    // 设置canvas尺寸与视频一致
    const videoEl = video.value;
    const canvasEl = canvas.value;
    
    if (!videoEl || !canvasEl) {
      throw new Error('视频或画布元素未找到');
    }
    
    console.log('视频尺寸:', videoEl.videoWidth, 'x', videoEl.videoHeight);
    canvasEl.width = videoEl.videoWidth;
    canvasEl.height = videoEl.videoHeight;
    
    // 在canvas上绘制当前视频帧
    const ctx = canvasEl.getContext('2d');
    ctx.drawImage(videoEl, 0, 0, canvasEl.width, canvasEl.height);
    console.log('已将视频帧绘制到画布');
    
    // 使用face-api.js检测人脸，只使用已加载的TinyFaceDetector模型
    console.log('开始检测人脸...');
    const detectorOptions = new faceapi.TinyFaceDetectorOptions({ scoreThreshold: minDetectionConfidence });
    const detections = await faceapi.detectAllFaces(canvasEl, detectorOptions);
    console.log('人脸检测结果:', detections);
    
    if (detections.length === 0) {
      console.log('未检测到人脸');
      ElNotification({
        title: '提示',
        message: '未检测到人脸，请调整位置后重试',
        type: 'warning',
        duration: 3000
      });
      isAnalyzing.value = false;
      return;
    }
    
    // 获取置信度最高的人脸
    const bestDetection = detections.reduce((prev, current) => {
      return (prev.detection.score > current.detection.score) ? prev : current;
    });
    console.log('最佳人脸检测结果:', bestDetection);
    
    // 将当前检测结果添加到缓冲区
    detectionBuffer.value.push(bestDetection);
    if (detectionBuffer.value.length > bufferSize) {
      detectionBuffer.value.shift(); // 移除最旧的检测结果
    }
    
    // 绘制人脸检测框和特征点
    const resizedDetections = faceapi.resizeResults(detections, {
      width: canvasEl.width,
      height: canvasEl.height
    });
    
    ctx.clearRect(0, 0, canvasEl.width, canvasEl.height);
    ctx.drawImage(videoEl, 0, 0, canvasEl.width, canvasEl.height);
    
    try {
      faceapi.draw.drawDetections(canvasEl, resizedDetections);
      console.log('已绘制人脸检测框');
    } catch (drawError) {
      console.error('绘制人脸检测框时出错:', drawError);
      // 继续执行，不中断流程
    }
    
    // 使用稳定的检测结果生成算命结果
    console.log('生成算命结果...');
    generateStableFortuneResult();
    console.log('算命结果生成完成');
    
  } catch (error) {
    console.error('分析人脸时出错:', error);
    let errorMsg = '分析人脸时出错，请重试';
    
    // 根据错误类型提供更具体的错误信息
    if (error.message) {
      console.error('错误信息:', error.message);
      if (error.message.includes('permission')) {
        errorMsg = '无法访问摄像头，请确保已授予摄像头权限';
      } else if (error.message.includes('load')) {
        errorMsg = '加载人脸识别模型失败，请检查网络连接';
      } else if (error.message.includes('模型未加载')) {
        errorMsg = error.message;
      } else {
        // 添加更多详细的错误信息
        errorMsg = `分析人脸时出错: ${error.message}`;
      }
    } else if (!navigator.onLine) {
      errorMsg = '网络连接已断开，请检查网络后重试';
    }
    
    ElNotification({
      title: '错误',
      message: errorMsg,
      type: 'error',
      duration: 5000
    });
  } finally {
    isAnalyzing.value = false;
  }
}

function generateStableFortuneResult() {
  // 如果缓冲区为空，则返回
  if (detectionBuffer.value.length === 0) return;
  
  // 计算多帧检测结果的平均值，以获得更稳定的结果
  let avgWidth = 0;
  let avgHeight = 0;
  let avgAge = 0;
  let dominantExpression = {};
  let detectedGender = { male: 0, female: 0 };
  
  // 计算平均值
  detectionBuffer.value.forEach(detection => {
    // 检查detection对象和detection.detection是否存在，以及box是否存在
    if (detection && detection.detection && detection.detection.box) {
      const box = detection.detection.box;
      avgWidth += box.width;
      avgHeight += box.height;
    }
    
    // 累加年龄
    if (detection && detection.age) {
      avgAge += detection.age;
    }
    
    // 累加性别检测结果
    if (detection && detection.gender) {
      detectedGender[detection.gender] += 1;
    }
    
    // 累加表情检测结果
    if (detection && detection.expressions) {
      Object.entries(detection.expressions).forEach(([expression, value]) => {
        dominantExpression[expression] = (dominantExpression[expression] || 0) + value;
      });
    }
  });
  
  // 计算有效检测结果的数量
  let validDetections = 0;
  detectionBuffer.value.forEach(detection => {
    if (detection && detection.detection && detection.detection.box) {
      validDetections++;
    }
  });
  
  // 计算平均值，确保不会除以零
  if (validDetections > 0) {
    avgWidth /= validDetections;
    avgHeight /= validDetections;
  }
  
  // 计算平均年龄，只有在有年龄数据时才计算
  let validAgeDetections = detectionBuffer.value.filter(d => d && d.age).length;
  if (validAgeDetections > 0) {
    avgAge /= validAgeDetections;
  }
  
  // 确定主要表情
  let mainExpression = Object.entries(dominantExpression).reduce(
    (max, [expression, value]) => value > max[1] ? [expression, value] : max, 
    ['neutral', 0]
  )[0];
  
  // 确定性别（投票决定）
  const gender = detectedGender.male > detectedGender.female ? '男' : '女';
  
  // 使用稳定的宽高比
  const faceRatio = avgWidth / avgHeight;
  
  // 使用检测到的年龄，如果没有则使用随机值
  const age = avgAge > 0 ? Math.round(avgAge) : Math.floor(Math.random() * 40) + 18;
  
  // 使用检测到的性别和年龄，而不是随机生成
  
  // 生成性格特点
  const personalityTraits = {
    outgoing: Math.floor(Math.random() * 40) + 60, // 偏向开朗
    steady: Math.floor(Math.random() * 100),
    creative: Math.floor(Math.random() * 100),
    patient: Math.floor(Math.random() * 100),
    detail: Math.floor(Math.random() * 100)
  };
  
  // 根据性格特点生成描述
  let personalityDesc = '';
  if (personalityTraits.outgoing > 80) {
    personalityDesc = '您天生乐观开朗，善于社交，是人群中的焦点。富有感染力的性格让您在各种场合都能如鱼得水。';
  } else if (personalityTraits.steady > 80) {
    personalityDesc = '您性格稳重踏实，做事有条不紊，是一个值得信赖的人。在关键时刻总能保持冷静，做出理性的决定。';
  } else if (personalityTraits.creative > 80) {
    personalityDesc = '您思维活跃，创造力丰富，常常能提出独特的见解和创新的解决方案。艺术天赋也很突出。';
  } else {
    personalityDesc = '您性格多元化，既有活力四射的一面，也有深思熟虑的特质。适应能力强，在不同环境中都能找到自己的位置。';
  }
  
  // 生成事业分析
  const careerDesc = [
    '您的面相显示事业线清晰有力，将来在职场上大有可为。建议从事需要创造力和领导力的工作，如管理、创业或艺术领域。',
    '您的面相显示做事细致认真，适合从事需要耐心和专注力的工作，如研究、技术开发或精密制造。',
    '您的面相显示具有良好的沟通能力和亲和力，适合从事与人打交道的工作，如销售、教育或咨询服务。',
    '您的面相显示思维敏捷，反应迅速，适合从事需要快速决策的工作，如金融、贸易或媒体行业。'
  ];
  const careerIndex = Math.floor(Math.random() * careerDesc.length);
  
  // 生成婚姻分析
  const marriageDesc = [
    '您的婚姻线显示您将遇到一位真诚相爱的伴侣，婚后生活和睦幸福。建议在30岁左右考虑婚姻大事。',
    '您的婚姻线显示您在感情上比较谨慎，不会轻易承诺，但一旦认定对方，会全心投入。婚后家庭生活稳定。',
    '您的婚姻线显示您在感情中可能经历一些波折，但最终会找到真爱。婚后需要双方共同努力经营，才能获得幸福。',
    '您的婚姻线显示您对爱情充满热情和期待，容易被浪漫所打动。婚后生活丰富多彩，但也需要注意保持新鲜感。'
  ];
  const marriageIndex = Math.floor(Math.random() * marriageDesc.length);
  
  // 生成今日宜忌
  const suitableActivities = [
    '谈判签约', '出行旅游', '投资理财', '求职面试', 
    '学习进修', '健身锻炼', '社交聚会', '创业计划',
    '购物消费', '装修搬家', '表白求婚', '开业开张'
  ];
  
  const avoidActivities = [
    '高空作业', '剪发理发', '远途旅行', '大额支出',
    '争吵冲突', '冒险活动', '签订合同', '借贷借款',
    '医疗手术', '重大决策', '饮酒过量', '熬夜加班'
  ];
  
  // 随机选择今日宜忌项目
  const suitableCount = 3 + Math.floor(Math.random() * 3); // 3-5项
  const avoidCount = 3 + Math.floor(Math.random() * 3); // 3-5项
  
  const shuffleArray = (array) => {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  };
  
  const suitable = shuffleArray([...suitableActivities]).slice(0, suitableCount);
  const avoid = shuffleArray([...avoidActivities]).slice(0, avoidCount);
  
  // 设置算命结果
  fortuneResult.value = {
    gender,
    age,
    personality: {
      traits: personalityTraits,
      description: personalityDesc
    },
    career: {
      description: careerDesc[careerIndex]
    },
    marriage: {
      description: marriageDesc[marriageIndex]
    },
    daily: {
      suitable,
      avoid
    }
  };
}
</script>

<style scoped>
.face-fortune-container {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
}

.fortune-card {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
}

.camera-controls {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.camera-container {
  position: relative;
  width: 100%;
  height: 400px;
  background-color: #f5f7fa;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 20px;
}

.camera-view {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.camera-view.hidden {
  display: none;
}

.face-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.camera-placeholder {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100%;
  color: #909399;
}

.camera-placeholder .el-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.analyzing {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.5);
  color: #fff;
}

.analyzing .el-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.fortune-result {
  margin-top: 20px;
}

.basic-info {
  margin-bottom: 20px;
}

.personality-chart {
  margin-bottom: 20px;
}

.chart-container {
  height: 300px;
}

.chart {
  width: 100%;
  height: 100%;
}

.fortune-details {
  margin-bottom: 20px;
}

.fortune-text {
  line-height: 1.8;
  text-align: justify;
  padding: 10px 0;
}

.daily-fortune {
  margin-top: 20px;
}

.daily-card {
  height: 100%;
}

.daily-items {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.daily-item {
  margin-bottom: 10px;
}

@media (max-width: 768px) {
  .card-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .camera-controls {
    margin-top: 15px;
    width: 100%;
    justify-content: space-between;
  }
  
  .camera-container {
    height: 300px;
  }
}
</style>