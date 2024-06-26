<template>
	<div>
		<div style="position:relative">
			<div style="margin-left:10px;">
				<v-row>
					<div class="color-box-style" style="background-color:rgb(25,118,210); "></div>
					<v-row style="font-size:20px;">
						<div>목표수준 - </div>
						<div style="font-weight: 700;">&nbsp;Maturity Level:&nbsp;</div>
						<div style="color:#1976D2; font-weight: 700;">{{ slaResult.cloudStatus }},</div>
						<div style="font-weight: 700;">&nbsp;SLA:&nbsp;</div>
						<div style="color:#1976D2; font-weight: 700;">{{ slaResult.percentage }}</div>
					</v-row>
				</v-row>
				<v-row>
					<div class="color-box-style" style="background-color:rgba(255, 183, 77, 1); "></div>
					<div style="font-size:20px;">현수준</div>
				</v-row>
			</div>
		</div>
		<svg :width="chartWidth" :height="chartHeight">
			<g :transform="`translate(${chartCenterX}, ${chartCenterY})`">
				<g v-for="(perspective, index) in chartData.perspectives" :key="`perspective-${index}₩`">
					<line
						:x1="0"
						:y1="0"
						:x2="getCoordinate(chartRadius, index, chartData.perspectives.length)[0]"
						:y2="getCoordinate(chartRadius, index, chartData.perspectives.length)[1]"
						stroke="lightgray"
					/>
					<g v-for="level in maxDataValue" :key="`level-line-${index}-${level}`">
						<line v-if="level < maxDataValue"
							:x1="getLevelLineCoordinate(index, level)[0]"
							:y1="getLevelLineCoordinate(index, level)[1]"
							:x2="getLevelLineCoordinate(index, level)[2]"
							:y2="getLevelLineCoordinate(index, level)[3]"
							stroke="lightgray"
							stroke-width="2"
						/>
					</g>
					<text
						:x="getCoordinate(chartRadius + labelOffset, index, chartData.perspectives.length)[0]"
						:y="getCoordinate(chartRadius + labelOffset, index, chartData.perspectives.length)[1]"
						dominant-baseline="middle"
						text-anchor="middle"
					>
						{{ perspective.name }}
					</text>
				</g>
				<g>
					<polygon
						:points="getPolygonPoints(chartData.perspectives)"
						fill="rgba(255, 183, 77,0.2)"
						stroke="rgba(255, 183, 77, 1)"
					/>
					<polygon
						:points="getPolygonPointsGoal(chartData.perspectives)"
						fill="rgb(25, 118, 210, 0.2)"
            			stroke="rgb(25, 118, 210, 1)"
					/>
					<g v-for="(perspective, index) in chartData.perspectives">
						<circle
							:cx="getCoordinateForCircle(perspective, index)[0]"
							:cy="getCoordinateForCircle(perspective, index)[1]"
							:r="pointRadius"
							fill="rgba(255, 183, 77, 1)"
						/>
						<circle
							:cx="getCoordinateForCircleGoal(perspective, index)[0]"
							:cy="getCoordinateForCircleGoal(perspective, index)[1]"
							:r="pointRadius"
							fill="rgb(25, 118, 210, 1)"
						/>
					</g>
				</g>
			</g>
		</svg>
	</div>
</template>
  
<script>
import SLABase from './SLABase.vue'

export default {
    name: "SpiderChart",
	mixins: [
		SLABase
	],
	props: {
		selectedProfile: {
            type: Object,
            required: true
        },
		chartData: {
            type: Object,
            required: true
		},
		chartWidth: Number,
		chartHeight: Number,
		chartCenterX: Number,
		chartCenterY: Number,
		chartRadius: Number,
		labelOffset: Number,
		maxDataValue: Number,
		pointRadius: Number
	},
    comments: {
    },
    data () {
		return {
		}
    },
	created() {
		this.getSLAPercentage(this.chartData);
	},
	watch: {
		selectedProfile: {
            deep: true,
            handler() {
                this.getSLAPercentage(this.chartData);
            }
        },
    },
    methods: {
		getDataLength(perspectives) {
			let count = 0;
			Object.keys(perspectives).forEach(key => {
				var perspective = perspectives[key]
				perspective.details.forEach(detail => {
					count += 1;
				});
			});
			return count;
		},
		getCoordinateForCircle(perspective, index) {
			// 첫 번째로 true를 만나면 그 이후 모든 levels를 true로 간주
			let firstTrueFound = false;
			let completedLevels = 0;
			for (let i = perspective.levels.length - 1; i >= 0; i--) {
				const level = perspective.levels[i];
				const hasCheckedCheckpoint = level.checkpoints.some(checkpoint => checkpoint.checked);
				if (hasCheckedCheckpoint) {
					firstTrueFound = true;
				}
				if (firstTrueFound) {
					completedLevels++;
				}
			}
			const radius = this.chartRadius * (completedLevels / this.maxDataValue);
			return this.getCoordinate(radius, index, this.chartData.perspectives.length);
		},
		getCoordinateForCircleGoal(perspective, index) {
			const radius = this.chartRadius * (perspective.goalLevel / this.maxDataValue);
			return this.getCoordinate(radius, index, this.chartData.perspectives.length);
		},
		getCoordinate(radius, index, total) {
			const angle = (Math.PI * 2 * index) / total - Math.PI / 2;
			const x = radius * Math.cos(angle);
			const y = radius * Math.sin(angle);
			return [x, y];
		},
		getLevelLineCoordinate(index, level) {
			// 각도 계산
			const angle = (Math.PI * 2 * index) / this.chartData.perspectives.length - Math.PI / 2;
			// 해당 레벨에서의 반지름 계산
			const radius = this.chartRadius * (level / this.maxDataValue);
			// 축의 좌표를 계산
			const x = radius * Math.cos(angle);
			const y = radius * Math.sin(angle);

			// 수평 선의 길이를 정의
			const lineLength = 5; // 가로 선의 길이는 항상 일정

			// 축에 수직인 선의 끝점을 계산하기 위한 각도 조정
			const anglePerpendicular = angle + Math.PI / 2;

			// 수평 선의 시작점과 끝점 계산
			const x1 = x + lineLength * Math.cos(anglePerpendicular);
			const y1 = y + lineLength * Math.sin(anglePerpendicular);
			const x2 = x - lineLength * Math.cos(anglePerpendicular);
			const y2 = y - lineLength * Math.sin(anglePerpendicular);

			return [x1, y1, x2, y2];
		},
		getPolygonPoints(perspectives) {
			if (!perspectives || perspectives.length === 0) {
				return '';
			}
			var perspectiveArray = perspectives
				.map((perspective, index) => {
					// 첫 번째로 true를 만나면 그 이후 모든 levels를 true로 간주
					let firstTrueFound = false;
					let completedLevels = 0;
					for (let i = perspective.levels.length - 1; i >= 0; i--) {
						const level = perspective.levels[i];
						const hasCheckedCheckpoint = level.checkpoints.some(checkpoint => checkpoint.checked);
						if (hasCheckedCheckpoint) {
							firstTrueFound = true;
						}
						if (firstTrueFound) {
							completedLevels++;
						}
					}
					// 완료된 levels의 수에 따라 radius 계산
					const radius = this.chartRadius * (completedLevels / this.maxDataValue);
					return this.getCoordinate(radius, index, perspectives.length).join(',');
				})
				.join(' ');

			return perspectiveArray;
		},
		getPolygonPointsGoal(perspectives) {
			if (!perspectives || perspectives.length === 0) {
				return '';
			}
			var perspectiveArray = perspectives
			.map((perspective, index) => {
				const radius = this.chartRadius * (perspective.goalLevel / this.maxDataValue);
				return this.getCoordinate(radius, index, perspectives.length).join(',');
			})
			.join(' ');

			return perspectiveArray
		},
	}
}
</script>
  
<style>
</style>
  
  