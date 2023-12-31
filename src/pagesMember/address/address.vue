<script setup lang="ts">
	import { deleteMemberAddressByIdAPI, getMemberAddressAPI } from '@/services/address'
	import { useAddressStore } from '@/stores/modules/address'
	import type { AddressItem } from '@/types/address'
	import { onShow } from '@dcloudio/uni-app'
	import { ref } from 'vue'

	// 获取收货地址列表数据
	const addressList = ref<AddressItem[]>([])
	const getMemberAddressData = async () => {
		const res = await getMemberAddressAPI()
		addressList.value = res.data
	}

	// 初始化调用(页面显示)
	onShow(() => {
		getMemberAddressData()
	})

	// 删除收货地址
	const onDeleteAddress = (id : number) => {
		// 二次确认
		uni.showModal({
			content: '删除地址?',
			success: async (res) => {
				if (res.confirm) {
					// 根据id删除收货地址
					await deleteMemberAddressByIdAPI(id)
					// 重新获取收货地址列表
					getMemberAddressData()
				}
			},
		})
	}

	// 修改收货地址
	const onChangeAddress = (item : AddressItem) => {
		// 修改地址
		const addressStore = useAddressStore()
		addressStore.changeSelectedAddress(item)
		// 返回上一页
		uni.navigateBack()
	}
</script>

<template>
	<view class="viewport">
		<!-- 地址列表 -->
		<scroll-view enable-back-to-top class="scroll-view" scroll-y>
			<view v-if="addressList.length" class="address">
				<uni-swipe-action class="address-list">
					<!-- 收货地址项 -->
					<uni-swipe-action-item class="item" v-for="item in addressList" :key="item.id">
						<view class="item-content" @tap="onChangeAddress(item)">
							<view class="user">
								<text class="user-name">{{ item.receive_name }}</text>
								<text class="contact">{{ item.receive_tel }}</text>
								<text v-if="item.isDefault" class="badge">默认</text>
							</view>
							<view class="locate">{{ item.area_info }} {{ item.address }}</view>
						</view>
						<navigator class="edit" hover-class="none"
							:url="`/pagesMember/address-form/address-form?id=${item.id}`">
							<image src="../../static/icon/way_icon.png" class="list-icon" mode="widthFix"></image>
						</navigator>
						<!-- 右侧插槽 -->
						<template #right>
							<button @tap="onDeleteAddress(item.id)" class="delete-button">删除</button>
						</template>
					</uni-swipe-action-item>
				</uni-swipe-action>
			</view>
			<view v-else class="blank">
				<image class="blank-img" src="../../static/icon/none.png" mode="widthFix"></image>
				<view>您还未添加收获地址哦~</view>
			</view>
		</scroll-view>
		<!-- 添加按钮 -->
		<!-- <view class="add-btn" v-if="addressList.length == 0"> -->
		<view class="add-btn">
			<navigator hover-class="none" url="/pagesMember/address-form/address-form">
				新建地址
			</navigator>
		</view>
	</view>
</template>

<style lang="scss">
	page {
		height: 100%;
		overflow: hidden;
	}

	/* 删除按钮 */
	.delete-button {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 50px;
		height: 100%;
		font-size: 28rpx;
		color: #fff;
		border-radius: 0;
		padding: 0;
		background-color: #cf4444;
	}

	.viewport {
		display: flex;
		flex-direction: column;
		height: 100%;
		background-color: #f4f4f4;

		.scroll-view {
			padding-top: 20rpx;
		}
	}

	.address {
		padding: 0 20rpx;
		margin: 0 30rpx;
		border-radius: 10rpx;
		background-color: #fff;

		.item-content {
			line-height: 1;
			padding: 40rpx 10rpx 38rpx;
			border-bottom: 1rpx solid #ddd;
			position: relative;
		}

		.edit {
			position: absolute;
			top: 36rpx;
			right: 10rpx;
			padding: 2rpx 0 2rpx 20rpx;
			font-size: 26rpx;
			color: #666;
			line-height: 1;

			.list-icon {
				width: 30rpx;
			}
		}

		.item:last-child .item-content {
			border: none;
		}

		.user {
			font-size: 28rpx;
			margin-bottom: 20rpx;
			color: #333;

			.user-name {
				font-weight: 700;
				padding-right: 10rpx;
			}

			.contact {
				color: #666;
			}

			.badge {
				display: inline-block;
				padding: 4rpx 10rpx 2rpx 14rpx;
				margin: 2rpx 0 0 10rpx;
				font-size: 26rpx;
				color: #27ba9b;
				border-radius: 6rpx;
				border: 1rpx solid #27ba9b;
			}
		}

		.locate {
			line-height: 1.6;
			font-size: 26rpx;
			color: #333;
		}
	}

	.blank {
		margin-top: 300rpx;
		text-align: center;
		font-size: 28rpx;
		color: #999;
		.blank-img {
			width: 50%;
		}
	}

	.add-btn {
		height: 100rpx;
		line-height: 100rpx;
		margin: 30rpx 60rpx;
		color: #fff;
		border-radius: 80rpx;
		font-size: 30rpx;
		text-align: center;
		background-image: linear-gradient(to right, #575759, #212123);
	}
</style>