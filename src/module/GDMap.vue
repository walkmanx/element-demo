<！-- 创建高德地图Demo -- >
<template>
	<div>
		<div>
			<el-button type="primary" @click="initMap">加载地图</el-button>
			<el-button type="primary" @click="addPlugin">引入工具条插件</el-button>
			<el-button type="primary" @click="destroyMap">销毁地图</el-button>
			<el-button type="primary" @click="setLocation">定位当前位置</el-button>
			<el-button type="primary" @click="addDefaultMarker">添加点标记</el-button>
			<el-button type="primary" @click="removeMarker">移除点标记</el-button>
			<el-button type="primary" @click="loadWeather">加载天气查询插件</el-button>
			<el-button type="primary" @click="getBrowserInfo">获取浏览器信息</el-button>
		</div>
		<div id="container"></div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				map: null,
				markerList: []
			}
		},
		mounted() {
			// this.initMap()
		},
		methods: {
			// 初始化map
			initMap() {
				const self = this;
				this.map = new AMap.Map('container', {
					//center: [108.91083, 34.23936], // 设置地图中心点坐标，如果不设置的话则默认显示当前城市
					zoom: 11, //设置地图显示的缩放级别
					//pitch:75, // 地图俯仰角度，有效范围 0 度- 83 度
					//viewMode:'3D', // 地图模式
					//mapStyle: 'amap://styles/dark', //设置地图的显示样式
					lang: 'zh_cn', //设置地图语言类型
				});

				// 同时引入工具条插件，比例尺插件和鹰眼插件
				//this.addPlugin();

				this.map.on('complete', function() {
					// 地图图块加载完成后触发
					self.$message({
						message: '地图加载完成，当前地图中心点为：' + self.map.getCenter(),
						type: 'success'
					});
				});


				this.map.on('click', function(ev) {
					// 触发事件的对象
					var target = ev.target;
					// 触发事件的地理坐标，AMap.LngLat 类型
					var lnglat = ev.lnglat;
					// 触发事件的像素坐标，AMap.Pixel 类型
					var pixel = ev.pixel;
					// 触发事件类型
					var type = ev.type;


					self.$message('您在 [ ' + ev.lnglat.getLng() + ',' + ev.lnglat.getLat() + ' ] 的位置单击了地图！');

				});
			},
			// 销毁地图
			destroyMap() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				this.map.destroy();
			},
			// 添加工具条插件
			addPlugin() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				const self = this;
				AMap.plugin([
					'AMap.ToolBar',
					'AMap.Scale',
					'AMap.OverView',
					'AMap.MapType',
					'AMap.Geolocation',
				], function() {
					// 在图面添加工具条控件，工具条控件集成了缩放、平移、定位等功能按钮在内的组合控件
					self.map.addControl(new AMap.ToolBar());

					// 在图面添加比例尺控件，展示地图在当前层级和纬度下的比例尺
					self.map.addControl(new AMap.Scale());

					// 在图面添加鹰眼控件，在地图右下角显示地图的缩略图
					self.map.addControl(new AMap.OverView({
						isOpen: true
					}));

					// 在图面添加类别切换控件，实现默认图层与卫星图、实施交通图层之间切换的控制
					self.map.addControl(new AMap.MapType());

					// 在图面添加定位控件，用来获取和展示用户主机所在的经纬度位置
					self.map.addControl(new AMap.Geolocation());
				});
			},
			// 创建点标记
			createMarker(x, y, title) {
				var marker = new AMap.Marker({
					position: new AMap.LngLat(x, y), // 经纬度对象，也可以是经纬度构成的一维数组[116.39, 39.9]
					title: title
				});
				return marker;
			},
			// 添加默认点标记
			addDefaultMarker() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				// 创建点标记
				this.marker = this.createMarker(108.91083, 34.23936, '我的位置');
				// 将创建的点标记添加到已有的地图实例：
				this.map.add(this.marker);

			},
			// 添加点标记
			addMarker(x, y, title) {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				// 创建点标记
				this.marker = this.createMarker(x, y, title);
				// 将创建的点标记添加到已有的地图实例：
				this.map.add(this.marker);

			},
			// 移除点标记
			removeMarker() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}

				if (!this.marker) {
					this.$message.warning('请先创建点标记');
					return;
				}
				this.map.remove(this.marker);
				this.map.setFitView();
				this.marker = null;
			},
			// 加载天气查询插件
			loadWeather() {
				var self = this;
				AMap.plugin('AMap.Weather', function() {
					//创建天气查询实例
					var weather = new AMap.Weather();

					//执行实时天气信息查询
					weather.getLive('西安市', function(err, data) {
						self.$message(JSON.stringify(data));
						console.log(JSON.stringify(data));
					});
				});
			},
			// 浏览器定位
			setLocation() {
				const self = this;
				AMap.plugin('AMap.Geolocation', function() {
					var geolocation = new AMap.Geolocation({
						// 是否使用高精度定位，默认：true
						enableHighAccuracy: true,
						// 设置定位超时时间，默认：无穷大
						timeout: 10000,
						// 定位按钮的停靠位置的偏移量，默认：Pixel(10, 20)
						buttonOffset: new AMap.Pixel(10, 20),
						//  定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
						zoomToAccuracy: true,
						//  定位按钮的排放位置,  RB表示右下
						buttonPosition: 'RB'
					})

					geolocation.getCurrentPosition()
					AMap.event.addListener(geolocation, 'complete', onComplete)
					AMap.event.addListener(geolocation, 'error', onError)

					function onComplete(data) {
						// data是具体的定位信息
						console.log(JSON.stringify(data));
						self.addMarker(data.position.lng, data.position.lat, data.formattedAddress);

						self.map.setFitView();
						// self.$message(JSON.stringify(data));
					}

					function onError(data) {
						// 定位出错
						self.$message(JSON.stringify(data));
					}
				})
			},
			// 判断浏览器、系统版本，以及浏览器的一些绘制特性
			getBrowserInfo() {
				// 浏览器 UA 信息
				var ua = AMap.Browser.ua;
				// 浏览器平台类型
				var plat = AMap.Browser.plat;
				// 浏览器是否支持 Canvas
				var canvas = AMap.Browser.isCanvas;
				// 浏览器是否支持 WebGL
				var isWebGL = AMap.Browser.isWebGL();

				console.log('浏览器 UA 信息' + ua);
				console.log('浏览器平台类型' + plat);
				console.log('浏览器是否支持 Canvas' + canvas);
				console.log('浏览器是否支持 WebGL' + isWebGL);
			},
			// 距离、长度、面积计算 AMap.GeometryUtil
			computerFunc(){
				// 计算两点间的实际距离 AMap.GeometryUtil.distance
				var p1 = [116.434027, 39.941037];
				var p2 = [116.461665, 39.941564];
				// 返回 p1 到 p2 间的地面距离，单位：米
				var dis = AMap.GeometryUtil.distance(p1, p2);
				// 计算点到线段的最短距离 AMap.GeometryUtil.distanceToSegment
				var p0 = [116.450378, 39.947585];
				var p1 = [116.434027, 39.941037];
				var p2 = [116.461665, 39.941564];
				// 返回 p0 到线段 p1-p2 的最短地面距离，单位：米
				var dis = AMap.GeometryUtil.distanceToSegment(p0, p1, p2);
				// 计算点到路径的最短距离 AMap.GeometryUtil.distanceToLine
		
				// 计算路径的实际长度 AMap.GeometryUtil.distanceOfLine
				var p0 = [116.450378, 39.947585];
				var p1 = [116.434027, 39.941037];
				var p2 = [116.461665, 39.941564];
				// 返回线段 p0-p1-p2 的实际长度，单位：米
				var dis = AMap.GeometryUtil.distanceOfLine([p0, p1, p2]);
				// 计算封闭区域的面积 AMap.GeometryUtil.ringArea
				var p0 = [116.450378, 39.947585];
				var p1 = [116.434027, 39.941037];
				var p2 = [116.461665, 39.941564];
				// 返回点 p0-p1-p2 围成的闭合区域面积，单位：平方米
				var area = AMap.GeometryUtil.ringArea([p0, p1, p2]);
				
			}
		}
	}
</script>

<style>
	#container {
		width: 1200px;
		height: 800px;
		top: 20px;
	}
</style>
