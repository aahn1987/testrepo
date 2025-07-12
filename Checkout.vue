<template>
	<div class="checkout-page">
		<breadcrumb :paths="[
			{ name: 'Home', link: '/' },
			{ name: 'checkout', link: null },
		]"></breadcrumb>

		<!-- loading -->
		<div v-if="isLoading && ($store.getters.cartIsEmpty || $store.getters.offerCartIsEmpty )  ">
			<div style="min-height: 450px; display: flex; justify-content: center; align-items: center">
				<layout-loading></layout-loading>
			</div>
		</div>

		<!-- error -->
		<div v-else-if="error.thereAnError" class="container-xl pt-3">
			<div class="text-center py-5 mt-5">
				<img width="175" class="mb-3 tw-mx-auto" src="/images/contact-us.png" alt="">
				<h3 class="mb-3">{{ $t("web._YOU_CANNOT_PROCEED_WITH_CHECKOUT") }}</h3>
				<p class="tw-text-xl tw-text-neutral-2 mx-auto tw-mb-4" style="max-width: 640px">
					{{ $t("web._SORRY_YOU_MIGHT_NOT_BE_ABLE_TO_PROCEED_WITH_YOUR_CHECKOUT") }}
				</p>
				<div class="alert alert-warning" role="alert">
					{{ $t("web._ERROR_CODES") }}:
					<span v-text="error.message"></span>
				</div>

				<div class="d-flex gap-2 justify-content-center">
					<a role="button" class="tw-outline-btn" @click="$router.go()">
						{{ $t("web._TRY_AGAIN") }}
					</a>
					<a href="https://ucare.umggroup.com/" role="button" class="tw-btn" target="_blank">
						{{ $t("web._CONTACT_US") }}
					</a>
				</div>

			</div>
		</div>

		<!-- page -->
		<div v-else>
			<div class="container-xl pt-3" v-if="!isLoading && !isQualifiedForShopping">
				<div class="text-center py-5 mt-5">
					<img width="175" class="mb-3" src="/images/contact-us.png" alt="">
					<h3 class="mb-3">{{ $t("web._YOU_CANNOT_PROCEED_WITH_CHECKOUT") }}</h3>
					<p class="standard-text regular-weight mx-auto" style="max-width: 640px">
						{{ $t("web._SORRY_YOU_MIGHT_NOT_BE_ABLE_TO_PROCEED_WITH_YOUR_CHECKOUT") }}
					</p>
					<div class="alert alert-warning" role="alert">
						{{ $t("web._ERROR_CODES") }}:
						<span v-for="(error, index) in errorArray">
							<span v-text="$t('web.' + convertToLangKey(error.code))"></span>
							<span v-if="index < errorArray.length - 1">, </span>
							<span v-else>. </span>
						</span>
					</div>

					<a href="https://ucare.umggroup.com/" role="button" class="btn un-md-primary-btn px-4 mt-3" target="_blank">
						{{ $t("web._CONTACT_US") }}
					</a>
				</div>
			</div>

			<div class="container-xl pt-3" v-else-if="needIrRegisteration">
				<div class="text-center py-5 mt-5">
					<h3 class="mb-3">{{ $t("web._YOU_CANNOT_PROCEED_WITH_CHECKOUT") }}</h3>
					<p v-if="parseInt($store.state.storeCurrencyTypeId) === 2" class="standard-text regular-weight mx-auto"
						style="max-width: 640px">
						{{ $t("web._Sorry_your_account_is_not_activated") }}
					</p>
					<p v-if="parseInt($store.state.storeCurrencyTypeId) === 1" class="standard-text regular-weight mx-auto"
						style="max-width: 640px">
						{{ $t("web._Sorry_your_account_is_not_activated_point_store") }}
					</p>
					<div class="alert alert-warning" role="alert">
						{{ $t("web._ERROR_CODES") }}:
					</div>

					<a v-if="parseInt($store.state.storeCurrencyTypeId) === 2" role="button" class="tw-btn  hover:tw-text-dark px-4 mt-3"
						@click="addRegisterationProduct()">
						{{ $t("web._ADD_REGISTERATION_PRODUCT") }}
						<i class="bx bx-loader bx-spin font-size-16 align-middle me-2" v-if="addRegisterationProductLoading
							"></i>
					</a>
				</div>
			</div>

			<div class="container-xl pt-3" v-else-if="$store.getters.cartIsEmpty && $store.getters.offerCartIsEmpty">
				<div class="tw-flex tw-flex-col tw-items-center" >
					<div class="">
						<img style="tw-mx-auto" src="/images/empty_cart.png" alt="" />
					</div>
					<div class="tw-my-3 tw-text-base" v-text="$t('web._SHOPPING_CART_IS_EMPTY')"></div>

					<router-link
						:class="{'disabled': $store.state.productsIsLoading}"
						to="/products"
						type="button"
						class="tw-outline-btn"
					>
						{{ $t("web._SHOP_NOW") }}
					</router-link>
				</div>
			</div>

			<div v-else>
				<!-- steps -->
				<div class="container-xl tw-mt-8">
					<!-- loading layout -->
					<div v-if="isLoading" class="row">
						<div class="col-lg-9 col-12 placeholder-glow">
							<div class="placeholder w-100" style="height: 300px"></div>
						</div>
						<div class="col-lg-3 col-12 placeholder-glow">
							<div class="placeholder w-100" style="height: 450px"></div>
						</div>
					</div>
					<div v-else class="row flex-lg-row g-3">
						<div class="col-lg-9">
							<!--  Checkout step -->
							<div class="tw-mt-4">
								<div class="accordion" id="accordionPanelsStayOpenExample">
									<!-- Address Accordion -->
									<div id="address-acc" class="accordion-item tw-mb-4 !tw-rounded-none !tw-border-1 !tw-border-gray/40">
										<div class="accordion-header tw-py-4 tw-px-5" :class="selectedAddress ? isRTL()?'tw-border-r-4 tw-border-solid tw-border-success':'tw-border-l-4 tw-border-solid tw-border-success' :''">
											<button
												class="accordion-button !tw-shadow-none !tw-drop-shadow-none !tw-border-none !tw-text-neutral-1 !tw-bg-light tw-py-0 tw-px-0"
												type="button"
												:class="{ 'tw-opacity-60':unqualifiedProducts }"
												:disabled=" unqualifiedProducts"
												data-bs-toggle="collapse"
												data-bs-target="#panelsStayOpen-collapseOne"
												aria-expanded="true"
												aria-controls="panelsStayOpen-collapseOne"
											>
												<div class="tw-flex tw-items-center">
													<svg v-if="selectedAddress" width="24" height="24" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
														<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15Z" fill="white"/>
														<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15ZM19.5125 12.7325C19.5875 12.6326 19.6418 12.5186 19.6721 12.3974C19.7025 12.2762 19.7083 12.1502 19.6892 12.0267C19.6701 11.9032 19.6266 11.7848 19.5611 11.6784C19.4956 11.572 19.4095 11.4797 19.3078 11.4071C19.2062 11.3344 19.091 11.2828 18.9691 11.2553C18.8473 11.2278 18.7211 11.2249 18.5981 11.2468C18.4751 11.2688 18.3577 11.3151 18.2528 11.383C18.148 11.451 18.0578 11.5392 17.9875 11.6425L13.9425 17.305L11.9125 15.275C11.7348 15.1094 11.4997 15.0192 11.2568 15.0235C11.014 15.0278 10.7822 15.1262 10.6105 15.298C10.4387 15.4697 10.3403 15.7015 10.336 15.9443C10.3317 16.1872 10.4219 16.4223 10.5875 16.6L13.4 19.4125C13.4962 19.5087 13.6123 19.5827 13.74 19.6296C13.8677 19.6764 14.0041 19.6948 14.1397 19.6837C14.2753 19.6725 14.4068 19.6319 14.5252 19.5648C14.6435 19.4977 14.7458 19.4056 14.825 19.295L19.5125 12.7325Z" fill="#25B77E"/>
													</svg>
													<svg v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 12 15" fill="none">
														<path fill-rule="evenodd" clip-rule="evenodd" d="M5.71181 14.2757L5.75859 14.3023L5.7773 14.313C5.8515 14.353 5.93453 14.374 6.01889 14.374C6.10324 14.374 6.18627 14.353 6.26047 14.313L6.27918 14.303L6.32663 14.2757C6.58801 14.1211 6.84305 13.956 7.09115 13.781C7.73342 13.3287 8.33352 12.8195 8.88416 12.2597C10.1833 10.933 11.5326 8.93967 11.5326 6.375C11.5326 4.91631 10.9517 3.51736 9.91775 2.48591C8.8838 1.45446 7.48145 0.875 6.01922 0.875C4.55699 0.875 3.15464 1.45446 2.12069 2.48591C1.08673 3.51736 0.505859 4.91631 0.505859 6.375C0.505859 8.939 1.8558 10.933 3.15428 12.2597C3.70471 12.8195 4.30458 13.3286 4.94662 13.781C5.19493 13.9561 5.4502 14.1211 5.71181 14.2757ZM6.01922 8.375C6.55094 8.375 7.06088 8.16429 7.43687 7.78921C7.81285 7.41414 8.02408 6.90543 8.02408 6.375C8.02408 5.84457 7.81285 5.33586 7.43687 4.96079C7.06088 4.58571 6.55094 4.375 6.01922 4.375C5.4875 4.375 4.97755 4.58571 4.60157 4.96079C4.22559 5.33586 4.01436 5.84457 4.01436 6.375C4.01436 6.90543 4.22559 7.41414 4.60157 7.78921C4.97755 8.16429 5.4875 8.375 6.01922 8.375Z" fill="#777777"/>
													</svg>
													<span class="tw-text-lg tw-font-bold tw-mx-2">{{ $t("web._ADDRESS") }}</span>
												</div>
											</button>
											<div v-if="selectedAddress" class="tw-mt-2">
												<div class="tw-text-header tw-text-base ">{{selectedAddress.city}}</div>
												<div class="tw-text-header tw-text-base ">{{selectedAddress.unit_address}}</div>
												<div class="d-flex tw-text-header tw-text-base ">
													{{selectedAddress.phone_number}}
												</div>
											</div>
										</div>
										<div id="panelsStayOpen-collapseOne" class="accordion-collapse collapse show" data-bs-parent="#accordionPanelsStayOpenExample">
											<div class="accordion-body">
												<div class="tw-bg-muted-light tw-px-3 tw-py-6 tw-flex tw-flex-col tw-gap-8">
													<div v-if="AddressisLoading" class="tw-flex tw-flex-wrap">
														<div v-for="index in 3" class="lg:tw-w-1/3 sm:tw-w-1/2 tw-w-full tw-px-2">
															<div class="placeholder-glow">
																<div class="placeholder w-100" style="height: 200px"></div>
															</div>
														</div>
													</div>
													<div v-else-if="addresses.length == 0" class="text-center">
														<img width="100" class="tw-mx-auto" src="/images/no-address-yet.png" alt="">
														<h3 class="tw-my-3 tw-text-xl">{{ $t("web._NO_ADDRESS_YET") }}</h3>
														<div class="tw-flex tw-justify-center text-center tw-py-0 tw-px-3">
															<button type="button"  class="tw-btn sm:tw-w-52 tw-w-full" @click.prevent="createAddressModal">
																{{ $t('web._ADD_NEW') }}
															</button>
														</div>
													</div>
													<div v-else>
														<div class="tw-flex tw-flex-wrap">
															<label
																v-for="(address, key) in addresses" :key="key"
																class="tw-text-dark lg:tw-w-1/3 sm:tw-w-1/2 tw-w-full tw-px-3 tw-mb-5"
																@click="() => {selectedAddress = address; fetchShippingTypes() ; MoveToNext(1) ;  }"
															>
																<div class="tw-w-full tw-h-full">
																	<div :class="selectedAddress == address && 'tw-border-success'" class="tw-relative tw-h-full tw-px-4 tw-py-3 tw-bg-light tw-border-gray tw-border d-flex tw-flex-col tw-justify-start tw-items-start tw-gap-1 tw-inline-flex">
																		<svg v-if="selectedAddress == address" :class="isRTL()?'tw-absolute tw-top-0 tw-left-0 -tw-translate-x-1/2 -tw-translate-y-1/2' :'tw-absolute tw-top-0 tw-right-0 tw-translate-x-1/2 -tw-translate-y-1/2'"  width="30" height="30" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
																			<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15Z" fill="white"/>
																			<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15ZM19.5125 12.7325C19.5875 12.6326 19.6418 12.5186 19.6721 12.3974C19.7025 12.2762 19.7083 12.1502 19.6892 12.0267C19.6701 11.9032 19.6266 11.7848 19.5611 11.6784C19.4956 11.572 19.4095 11.4797 19.3078 11.4071C19.2062 11.3344 19.091 11.2828 18.9691 11.2553C18.8473 11.2278 18.7211 11.2249 18.5981 11.2468C18.4751 11.2688 18.3577 11.3151 18.2528 11.383C18.148 11.451 18.0578 11.5392 17.9875 11.6425L13.9425 17.305L11.9125 15.275C11.7348 15.1094 11.4997 15.0192 11.2568 15.0235C11.014 15.0278 10.7822 15.1262 10.6105 15.298C10.4387 15.4697 10.3403 15.7015 10.336 15.9443C10.3317 16.1872 10.4219 16.4223 10.5875 16.6L13.4 19.4125C13.4962 19.5087 13.6123 19.5827 13.74 19.6296C13.8677 19.6764 14.0041 19.6948 14.1397 19.6837C14.2753 19.6725 14.4068 19.6319 14.5252 19.5648C14.6435 19.4977 14.7458 19.4056 14.825 19.295L19.5125 12.7325Z" fill="#25B77E"/>
																		</svg>
																		<div class="tw-flex tw-justify-between tw-w-full">
																			<div class="tw-text-base">{{address.unit_address}}</div>

																			<!-- <div class="tw-text-base  tw-text-header tw-flex tw-gap-1 tw-items-center">
																				<span class="header-txt-color tw-text-lg" >
																					<i :class="{'fas fa-star tw-text-warning': address.default_status == 1,'far fa-star': address.default_status != 1}"></i>
																				</span>
																			</div> -->
																		</div>

																		<div class="tw-text-base">{{address.city}}</div>
																		<div class="tw-text-base">{{address.unit_address}}</div>
																		<div class="d-flex tw-text-base">
																			<span>Phone: </span>
																			{{address.phone_number}}
																		</div>

																		<!-- <button v-if="selectedAddress == address" type="button" class="tw-outline-btn tw-w-full" @click="() => { selectedAddress = null ; selectedShipping = null  }">
																			{{ $t('web._UNSELECT') }}
																		</button>
																		<button v-else class="tw-outline-btn tw-w-full" @click="() => {selectedAddress = address; MoveToNext(1) ; fetchShippingTypes() }">
																			{{ $t('web._SELECT') }}
																		</button> -->

																	</div>
																</div>
															</label>

															<div class="tw-text-dark lg:tw-w-1/3 sm:tw-w-1/2 tw-w-full tw-px-3 tw-mb-5 tw-flex" style="min-height: 110px;">
																<button
																	class=" tw-border-dashed tw-border-gray tw-border-2 tw-w-full tw-h-full tw-flex tw-justify-center tw-items-center"
																	data-bs-toggle="modal"
																	data-bs-target=".bs-example-modal-center"
																	@click.prevent="createAddressModal"
																>
																	<i style="font-size: 28px;" class="fas fa-plus tw-text-neutral-2" ></i>
																</button>
															</div>
														</div>
														<!--
															element.style {
																position: fixed;
																bottom: 0;
																left: 0;
																right: 0;
																background: white;
																padding-block: 12px;
																z-index: 18;
																border-top: solid 1px gray;
															}
														-->
														<!-- <div class="tw-flex tw-justify-center text-center sm:tw-relative tw-fixed tw-bottom-0 tw-left-0 tw-right-0 sm:tw-bg-[#00000000] tw-bg-light sm:tw-py-0 tw-py-3 tw-z-20 sm:tw-border-t-0 tw-border-t-2 tw-border-gray/60 tw-px-3">
															<button type="button" :disabled="!selectedAddress" class="tw-btn sm:tw-w-52 tw-w-full" @click="MoveToNext(1)">
																{{ $t('web._NEXT') }}
															</button>
														</div> -->
													</div>
												</div>
											</div>
										</div>
									</div>
									<!-- End Address Accordion -->

									<!-- Pyament Accordion -->
									<div id="payment-acc" class="accordion-item tw-mb-4 !tw-rounded-none !tw-border-1 !tw-border-gray/40">
										<div class="accordion-header tw-py-4 tw-px-5" :class="selectedPayment ? isRTL()?'tw-border-r-4 tw-border-solid tw-border-success':'tw-border-l-4 tw-border-solid tw-border-success' :''">
											<button
												class="accordion-button !tw-shadow-none !tw-drop-shadow-none !tw-border-none !tw-text-neutral-1 !tw-bg-light tw-py-0 tw-px-0 collapsed"
												:class="{ 'tw-opacity-60': !selectedAddress || paymentIsRestricted || unqualifiedProducts }"
												:disabled="!selectedAddress || paymentIsRestricted || unqualifiedProducts"
												type="button"
												data-bs-toggle="collapse"
												data-bs-target="#panelsStayOpen-collapseTwo"
												aria-expanded="false"
												aria-controls="panelsStayOpen-collapseTwo"
											>
												<div class="tw-flex tw-items-center">
													<svg v-if="selectedPayment" width="24" height="24" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
														<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15Z" fill="white"/>
														<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15ZM19.5125 12.7325C19.5875 12.6326 19.6418 12.5186 19.6721 12.3974C19.7025 12.2762 19.7083 12.1502 19.6892 12.0267C19.6701 11.9032 19.6266 11.7848 19.5611 11.6784C19.4956 11.572 19.4095 11.4797 19.3078 11.4071C19.2062 11.3344 19.091 11.2828 18.9691 11.2553C18.8473 11.2278 18.7211 11.2249 18.5981 11.2468C18.4751 11.2688 18.3577 11.3151 18.2528 11.383C18.148 11.451 18.0578 11.5392 17.9875 11.6425L13.9425 17.305L11.9125 15.275C11.7348 15.1094 11.4997 15.0192 11.2568 15.0235C11.014 15.0278 10.7822 15.1262 10.6105 15.298C10.4387 15.4697 10.3403 15.7015 10.336 15.9443C10.3317 16.1872 10.4219 16.4223 10.5875 16.6L13.4 19.4125C13.4962 19.5087 13.6123 19.5827 13.74 19.6296C13.8677 19.6764 14.0041 19.6948 14.1397 19.6837C14.2753 19.6725 14.4068 19.6319 14.5252 19.5648C14.6435 19.4977 14.7458 19.4056 14.825 19.295L19.5125 12.7325Z" fill="#25B77E"/>
													</svg>
													<svg v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 16 16" fill="none">
														<path d="M8 5C7.60218 5 7.22064 5.15804 6.93934 5.43934C6.65804 5.72064 6.5 6.10218 6.5 6.5C6.5 6.89782 6.65804 7.27936 6.93934 7.56066C7.22064 7.84196 7.60218 8 8 8C8.39782 8 8.77936 7.84196 9.06066 7.56066C9.34196 7.27936 9.5 6.89782 9.5 6.5C9.5 6.10218 9.34196 5.72064 9.06066 5.43934C8.77936 5.15804 8.39782 5 8 5Z" fill="#777777"/>
														<path fill-rule="evenodd" clip-rule="evenodd" d="M1 3.25C1 2.55933 1.56 2 2.25 2H13.75C14.44 2 15 2.56 15 3.25V9.75C15 10.4407 14.44 11 13.75 11H2.25C2.08585 11 1.9233 10.9677 1.77165 10.9049C1.61999 10.842 1.48219 10.75 1.36612 10.6339C1.25004 10.5178 1.15797 10.38 1.09515 10.2284C1.03233 10.0767 1 9.91415 1 9.75V3.25ZM5.5 6.5C5.5 5.83696 5.76339 5.20107 6.23223 4.73223C6.70107 4.26339 7.33696 4 8 4C8.66304 4 9.29893 4.26339 9.76777 4.73223C10.2366 5.20107 10.5 5.83696 10.5 6.5C10.5 7.16304 10.2366 7.79893 9.76777 8.26777C9.29893 8.73661 8.66304 9 8 9C7.33696 9 6.70107 8.73661 6.23223 8.26777C5.76339 7.79893 5.5 7.16304 5.5 6.5ZM12.5 6C12.3674 6 12.2402 6.05268 12.1464 6.14645C12.0527 6.24022 12 6.36739 12 6.5V6.50533C12 6.78133 12.224 7.00533 12.5 7.00533H12.5053C12.6379 7.00533 12.7651 6.95265 12.8589 6.85889C12.9527 6.76512 13.0053 6.63794 13.0053 6.50533V6.5C13.0053 6.36739 12.9527 6.24022 12.8589 6.14645C12.7651 6.05268 12.6379 6 12.5053 6H12.5ZM3 6.5C3 6.36739 3.05268 6.24022 3.14645 6.14645C3.24021 6.05268 3.36739 6 3.5 6H3.50533C3.63794 6 3.76512 6.05268 3.85889 6.14645C3.95266 6.24022 4.00533 6.36739 4.00533 6.5V6.50533C4.00533 6.63794 3.95266 6.76512 3.85889 6.85889C3.76512 6.95265 3.63794 7.00533 3.50533 7.00533H3.5C3.36739 7.00533 3.24021 6.95265 3.14645 6.85889C3.05268 6.76512 3 6.63794 3 6.50533V6.5Z" fill="#777777"/>
														<path d="M1.5 12C1.36739 12 1.24021 12.0527 1.14645 12.1464C1.05268 12.2402 1 12.3674 1 12.5C1 12.6326 1.05268 12.7598 1.14645 12.8536C1.24021 12.9473 1.36739 13 1.5 13C5.1 13 8.58667 13.4813 11.9 14.3833C12.6933 14.5993 13.5 14.0113 13.5 13.17V12.5C13.5 12.3674 13.4473 12.2402 13.3536 12.1464C13.2598 12.0527 13.1326 12 13 12H1.5Z" fill="#777777"/>
													</svg>
													<span class="tw-text-lg tw-font-bold tw-mx-2">{{ $t("web._PAYMENT_METHOD") }}</span>
												</div>
											</button>
											<div v-if="selectedPayment" class="tw-flex md:tw-flex-row tw-flex-col md:tw-items-center tw-gap-3 tw-justify-between tw-mt-2 tw-py-2">

												<div class="tw-flex tw-flex-col tw-gap-1">
													<div class="tw-flex tw-items-center tw-gap-2 tw-mb-0">
														<div class="tw-text-lg tw-text-header">{{selectedPayment.title}}</div>
														<div class="tw-text-neutral-2 tw-text-sm">
															<span class="tw-custom-badge-success" v-if="selectedPayment.sale_fees_value">
																<span v-text="$t('web._extra_charge')"></span>
																<span v-if="parseInt(selectedPayment.sale_fees_calculation_type) === 1">
																	<span>% {{ selectedPayment.sale_fees_value }}</span>
																</span>
																<span v-else>
																	<span
																		v-text="numberFormat(selectedPayment.sale_fees_value_in_exchange_rate, $store.state.currentPriceCurrency)"></span>
																</span>
															</span>
															<span class="tw-custom-badge-success" v-else>{{ $t("web._NO_EXTRA_CHARGES") }}</span>
														</div>
													</div>

													<span
														class="regular-weight small-text text-muted tw-whitespace-normal"
														v-html="selectedPayment.description_translations[langsConvert[$i18n.locale]] || selectedPayment.description_translations['eng']"
													></span>
												</div>
												<div class="tw-flex tw-flex-wrap tw-gap-1 tw-min-h-[42px]" style="direction: ltr;">
													<div v-for="(card, index) in selectedPayment.payment_cards" :key="index">
														<img class="tw-h-[42px]" :src="getPaymentImageUrl(card)" alt="">
													</div>
												</div>

											</div>
										</div>
										<div id="panelsStayOpen-collapseTwo" class="accordion-collapse collapse" data-bs-parent="#accordionPanelsStayOpenExample">
											<div class="accordion-body">
												<div v-if="paymentMethodIsLoading" class="tw-flex tw-flex-wrap">
													<div v-for="index in 3" class="lg:tw-w-1/3 sm:tw-w-1/2 tw-w-full tw-px-2">
														<div class="placeholder-glow">
															<div class="placeholder w-100" style="height: 200px"></div>
														</div>
													</div>
												</div>
												<div class="">
													<!-- grouped -->
													<div v-for="(group, index) in groupedPaymentsArray" v-if="group.categorySlug != 'ungrouped'" :key="index" class="payment-group tw-flex tw-flex-col">
														<div class="tw-flex tw-items-center tw-gap-3 tw-h-[42px]">
															<input
																@input="() => (selectedPaymentGroup = group.categorySlug)"
																type="radio"
																:id="`payment-group-` + group.categorySlug"
																:ref="`payment-group-` + group.categorySlug"
																name="payment-group-1"
															>
															<label :for="`payment-group-` + group.categorySlug" class="tw-text-header tw-text-lg tw-mb-0">
																{{ group.categorySlug }}
															</label>
														</div>
														<div v-if="selectedPaymentGroup == group.categorySlug">
															<!-- Payment Methods under the Group -->
															<div class="tw-bg-muted-light tw-px-3 tw-mt-4 tw-pt-6 tw-flex tw-flex-wrap">
																<label
																	v-for="(item, index) in group.payments"
																	:key="index"
																	class="tw-text-dark lg:tw-w-1/3 sm:tw-w-1/2 tw-w-full tw-px-3 tw-mb-6 tw-h-auto"
																>
																	<div class="tw-flex tw-flex-col tw-gap-4 tw-h-full">
																		<div :class="selectedPayment == item && 'tw-border-success'" class="tw-relative tw-px-4 tw-py-3 tw-bg-light tw-border-gray tw-border tw-h-full tw-w-full tw-flex-col tw-justify-start tw-items-start tw-gap-1 tw-inline-flex">
																			<svg v-if="selectedPayment == item" :class="isRTL()?'tw-absolute tw-top-0 tw-left-0 -tw-translate-x-1/2 -tw-translate-y-1/2' :'tw-absolute tw-top-0 tw-right-0 tw-translate-x-1/2 -tw-translate-y-1/2'"  width="30" height="30" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
																				<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15Z" fill="white"/>
																				<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15ZM19.5125 12.7325C19.5875 12.6326 19.6418 12.5186 19.6721 12.3974C19.7025 12.2762 19.7083 12.1502 19.6892 12.0267C19.6701 11.9032 19.6266 11.7848 19.5611 11.6784C19.4956 11.572 19.4095 11.4797 19.3078 11.4071C19.2062 11.3344 19.091 11.2828 18.9691 11.2553C18.8473 11.2278 18.7211 11.2249 18.5981 11.2468C18.4751 11.2688 18.3577 11.3151 18.2528 11.383C18.148 11.451 18.0578 11.5392 17.9875 11.6425L13.9425 17.305L11.9125 15.275C11.7348 15.1094 11.4997 15.0192 11.2568 15.0235C11.014 15.0278 10.7822 15.1262 10.6105 15.298C10.4387 15.4697 10.3403 15.7015 10.336 15.9443C10.3317 16.1872 10.4219 16.4223 10.5875 16.6L13.4 19.4125C13.4962 19.5087 13.6123 19.5827 13.74 19.6296C13.8677 19.6764 14.0041 19.6948 14.1397 19.6837C14.2753 19.6725 14.4068 19.6319 14.5252 19.5648C14.6435 19.4977 14.7458 19.4056 14.825 19.295L19.5125 12.7325Z" fill="#25B77E"/>
																			</svg>
																			<div class="tw-text-base tw-whitespace-normal">{{item.title}}</div>
																			<span class="regular-weight small-text text-muted tw-whitespace-normal"
																				v-html="item.description_translations[langsConvert[$i18n.locale]] || item.description_translations['eng']">
																			</span>
																		</div>
																	</div>
																	<input
																		@input="() => (selectedPayment = item , MoveToNext(2))"
																		type="radio"
																		:id="`payment-` + item.id"
																		:ref="`payment-` + item.id"
																		name="group-1"
																		:disabled="item.disabled || (item.id == 19 && disabledOsvPaymentMethod)"
																		:checked="selectedPayment == item"
																		hidden
																	/>
																</label>
															</div>
														</div>
														<hr class="tw-border-b tw-border-gray/80 tw-my-3"/>
													</div>
													<!-- singal -->
													<div v-for="(group, index) in groupedPaymentsArray"  v-if="group.categorySlug == 'ungrouped'" :key="index" class="payment-group">
														<div v-for="(item, index) in group.payments" :for="`payment-group-` + item.id" :key="index">
															<label class="tw-flex md:tw-flex-row tw-flex-col md:tw-items-center tw-gap-3 tw-justify-between">
																<div :class="{'tw-opacity-40': item.disabled || (item.id == 19 && disabledOsvPaymentMethod)}" class="tw-flex tw-gap-3 tw-items-center">
																	<input
																		@input="() => (selectedPayment = item , selectedPaymentGroup = '' , MoveToNext(2))"
																		type="radio"
																		:id="`payment-group-` + item.id"
																		:ref="`payment-group-` + item.id"
																		name="payment-group-1"
																		:disabled="item.disabled || (item.id == 19 && disabledOsvPaymentMethod)"
																		:checked="selectedPayment == item"
																	/>
																	<div class="tw-flex tw-items-center tw-gap-2 tw-mb-0">
																		<span class="tw-text-lg tw-text-header"> {{ item.title }} </span>
																		<div class="tw-text-base tw-text-neutral-2">
																			<span class="tw-custom-badge-success" v-if="item.sale_fees_value">
																				<span v-if="parseInt(item.sale_fees_calculation_type) === 1">
																					<span>% {{ item.sale_fees_value }} - {{ numberFormat(calculatePaymentFees(item),$store.state.currentPriceCurrency) }}</span>
																				</span>
																				<span v-else>
																					{{numberFormat(item.sale_fees_value_in_exchange_rate, $store.state.currentPriceCurrency)}} - {{ numberFormat(calculatePaymentFees(item),$store.state.currentPriceCurrency) }}
																				</span>
																			</span>
																			<span v-else class="tw-custom-badge-success">
																				{{ $t("web._Free") }}
																			</span>
																		</div>
																	</div>
																</div>
																<div class="tw-flex tw-flex-wrap tw-gap-1 tw-min-h-[42px] " style="direction: ltr;">
																	<!-- <div v-for="(card, index) in item.payment_cards.slice(0, 3)" :key="index"> -->
																	<div v-for="(card, index) in item.payment_cards" :key="index">
																		<img class="tw-h-[42px]" :src="getPaymentImageUrl(card)" alt="">
																	</div>
																	<!-- <div v-if="item.payment_cards.length > 3" class="">
																		<img class="tw-h-[42px]" src="/images/other-payments.svg" alt="">
																	</div> -->
																</div>
															</label>
															<hr class="tw-border-b tw-border-gray/80 tw-my-3"/>
														</div>
													</div>
												</div>

											</div>
										</div>
									</div>
									<!-- End Pyament  Accordion-->

									<!-- Shipping Accordion -->
									<div id="shipping-acc" v-if="!hideAddressAndShipping" class="accordion-item tw-mb-4 !tw-rounded-none !tw-border-1 !tw-border-gray/40">
										<div class="accordion-header tw-py-4 tw-px-5" :class="selectedShipping ? isRTL()?'tw-border-r-4 tw-border-solid tw-border-success':'tw-border-l-4 tw-border-solid tw-border-success' :''">
											<button
												class="accordion-button !tw-shadow-none !tw-drop-shadow-none !tw-border-none !tw-text-neutral-1 !tw-bg-light tw-py-0 tw-px-0 collapsed"
												:disabled="(!selectedAddress && !selectedPayment) || unqualifiedProducts"
												:class="{ 'tw-opacity-60': (!selectedAddress && !selectedPayment) || unqualifiedProducts }"
												type="button"
												data-bs-toggle="collapse"
												data-bs-target="#panelsStayOpen-collapseThree"
												aria-expanded="false"
												aria-controls="panelsStayOpen-collapseThree"
											>
												<div class="d-flex tw-items-center">
													<svg v-if="selectedShipping" width="24" height="24" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
														<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15Z" fill="white"/>
														<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15ZM19.5125 12.7325C19.5875 12.6326 19.6418 12.5186 19.6721 12.3974C19.7025 12.2762 19.7083 12.1502 19.6892 12.0267C19.6701 11.9032 19.6266 11.7848 19.5611 11.6784C19.4956 11.572 19.4095 11.4797 19.3078 11.4071C19.2062 11.3344 19.091 11.2828 18.9691 11.2553C18.8473 11.2278 18.7211 11.2249 18.5981 11.2468C18.4751 11.2688 18.3577 11.3151 18.2528 11.383C18.148 11.451 18.0578 11.5392 17.9875 11.6425L13.9425 17.305L11.9125 15.275C11.7348 15.1094 11.4997 15.0192 11.2568 15.0235C11.014 15.0278 10.7822 15.1262 10.6105 15.298C10.4387 15.4697 10.3403 15.7015 10.336 15.9443C10.3317 16.1872 10.4219 16.4223 10.5875 16.6L13.4 19.4125C13.4962 19.5087 13.6123 19.5827 13.74 19.6296C13.8677 19.6764 14.0041 19.6948 14.1397 19.6837C14.2753 19.6725 14.4068 19.6319 14.5252 19.5648C14.6435 19.4977 14.7458 19.4056 14.825 19.295L19.5125 12.7325Z" fill="#25B77E"/>
													</svg>
													<svg  v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 16 16" fill="none">
														<path d="M2.25 3C1.55933 3 1 3.56 1 4.25V9H9V4.25C9 3.55933 8.44 3 7.75 3H2.25ZM9 10H1V11.75C1 12.44 1.56 13 2.25 13H2.5C2.5 12.4696 2.71071 11.9609 3.08579 11.5858C3.46086 11.2107 3.96957 11 4.5 11C5.03043 11 5.53914 11.2107 5.91421 11.5858C6.28929 11.9609 6.5 12.4696 6.5 13H8.5C8.63261 13 8.75979 12.9473 8.85355 12.8536C8.94732 12.7598 9 12.6326 9 12.5V10Z" fill="#777777"/>
														<path d="M5.5 13C5.5 12.7348 5.39464 12.4804 5.20711 12.2929C5.01957 12.1054 4.76522 12 4.5 12C4.23478 12 3.98043 12.1054 3.79289 12.2929C3.60536 12.4804 3.5 12.7348 3.5 13C3.5 13.2652 3.60536 13.5196 3.79289 13.7071C3.98043 13.8947 4.23478 14 4.5 14C4.76522 14 5.01957 13.8947 5.20711 13.7071C5.39464 13.5196 5.5 13.2652 5.5 13ZM10.5 4.50001C10.3674 4.50001 10.2402 4.55269 10.1464 4.64646C10.0527 4.74023 10 4.8674 10 5.00001V12.5C10 12.558 10.01 12.6133 10.028 12.6653C10.1111 12.1728 10.3755 11.7291 10.7692 11.4217C11.1629 11.1143 11.6574 10.9653 12.1554 11.0041C12.6534 11.0429 13.1189 11.2666 13.4603 11.6313C13.8016 11.996 13.9942 12.4752 14 12.9747C14.5687 12.858 15.0147 12.3513 14.976 11.7193C14.8269 9.27825 13.9615 6.93528 12.488 4.98334C12.3732 4.83241 12.225 4.71019 12.0549 4.62632C11.8848 4.54245 11.6976 4.49921 11.508 4.50001H10.5Z" fill="#777777"/>
														<path d="M13 13C13 12.7348 12.8946 12.4804 12.7071 12.2929C12.5196 12.1054 12.2652 12 12 12C11.7348 12 11.4804 12.1054 11.2929 12.2929C11.1054 12.4804 11 12.7348 11 13C11 13.2652 11.1054 13.5196 11.2929 13.7071C11.4804 13.8946 11.7348 14 12 14C12.2652 14 12.5196 13.8946 12.7071 13.7071C12.8946 13.5196 13 13.2652 13 13Z" fill="#777777"/>
													</svg>
													<span class="tw-text-lg tw-font-bold tw-mx-2">{{ $t("web._SHIPPING") }}</span>
												</div>
											</button>

											<div v-if="selectedShipping" class="tw-flex md:tw-flex-row tw-flex-col md:tw-items-center tw-gap-3 tw-justify-between tw-mt-2 tw-py-2">
												<div class="tw-flex tw-flex-col tw-gap-1">
													<div class="tw-flex tw-items-center tw-gap-2 tw-mb-0">
														<div class="tw-text-header tw-text-lg">{{ (selectedShipping.content_translations[langsConvert[$i18n.locale]])? selectedShipping.content_translations[langsConvert[$i18n.locale]].title : selectedShipping.content_translations['eng'].title }}</div>
														<div class="tw-text-neutral-2 tw-text-sm">
															<span class="tw-custom-badge-success">
																<!-- <span v-text="$t('web._extra_charge')"></span> -->
																<span v-if="parseInt(selectedShipping.shipping_fees_calculation_type) === 1">
																	<!--percentage-->
																	<span v-if="selectedShipping.shipping_fees_value > 0" >% {{ selectedShipping.shipping_fees_value }}</span>
																	<span v-else>{{ $t("web._NO_EXTRA_CHARGES") }}
																	</span>
																</span>
																<span v-else-if="parseInt(selectedShipping.shipping_fees_calculation_type) === 2">
																	<!--value-->
																	<span v-if="selectedShipping.shipping_fees_value_in_exchange_rate > 0"  v-text="numberFormat(selectedShipping.shipping_fees_value_in_exchange_rate, $store.state.currentPriceCurrency)">
																	</span>
																	<span v-else>{{ $t("web._NO_EXTRA_CHARGES") }}
																	</span>
																</span>
																<span v-else-if="parseInt(selectedShipping.shipping_fees_calculation_type) === 3">
																	<!--per kg-->
																	<span v-if="selectedShipping.shipping_weight_fees_value_in_exchange_rate > 0" v-text="numberFormat(selectedShipping.shipping_weight_fees_value_in_exchange_rate, $store.state.currentPriceCurrency)">
																	</span>
																	<span v-else>{{ $t("web._NO_EXTRA_CHARGES") }}
																	</span>
																</span>
															</span>
														</div>
													</div>

													<span
														class="tw-text-neutral-2 tw-whitespace-normal"
														style="direction: rtl"
														v-if="selectedShipping.content_translations[langsConvert[$i18n.locale]] ||  selectedShipping.content_translations['eng']"
														v-html="(selectedShipping.content_translations[langsConvert[$i18n.locale]])?selectedShipping.content_translations[langsConvert[$i18n.locale]].description :  selectedShipping.content_translations['eng'].description"
													></span>
												</div>

												<div v-if="selectedShipping.expected_days > 0" class="tw-text-sm tw-w-fit">
													<div class="tw-border-primary/60 tw-flex tw-items-center tw-border">
														<div class="tw-p-1 tw-bg-muted-light">
															<svg width="32" height="32" viewBox="0 0 18 18" class="tw-stroke-primary" fill="none" xmlns="http://www.w3.org/2000/svg">
																<path d="M6.18751 14.0629C6.18751 14.3612 6.06899 14.6474 5.85801 14.8584C5.64703 15.0693 5.36088 15.1879 5.06251 15.1879C4.76415 15.1879 4.478 15.0693 4.26702 14.8584C4.05604 14.6474 3.93751 14.3612 3.93751 14.0629M6.18751 14.0629C6.18751 13.7645 6.06899 13.4783 5.85801 13.2674C5.64703 13.0564 5.36088 12.9379 5.06251 12.9379C4.76415 12.9379 4.478 13.0564 4.26702 13.2674C4.05604 13.4783 3.93751 13.7645 3.93751 14.0629M6.18751 14.0629H10.6875M3.93751 14.0629H2.53126C2.30749 14.0629 2.09288 13.974 1.93464 13.8157C1.77641 13.6575 1.68751 13.4429 1.68751 13.2191V10.6879M10.6875 14.0629H12.375M10.6875 14.0629V10.6879M1.68751 10.6879V4.96161C1.68632 4.75624 1.76136 4.55772 1.89811 4.40449C2.03485 4.25126 2.22358 4.1542 2.42776 4.13211C4.92755 3.87263 7.44748 3.87263 9.94726 4.13211C10.371 4.17561 10.6875 4.53561 10.6875 4.96161V5.68011M1.68751 10.6879H10.6875M14.625 14.0629C14.625 14.3612 14.5065 14.6474 14.2955 14.8584C14.0845 15.0693 13.7984 15.1879 13.5 15.1879C13.2016 15.1879 12.9155 15.0693 12.7045 14.8584C12.4935 14.6474 12.375 14.3612 12.375 14.0629M14.625 14.0629C14.625 13.7645 14.5065 13.4783 14.2955 13.2674C14.0845 13.0564 13.7984 12.9379 13.5 12.9379C13.2016 12.9379 12.9155 13.0564 12.7045 13.2674C12.4935 13.4783 12.375 13.7645 12.375 14.0629M14.625 14.0629H15.4688C15.9345 14.0629 16.3155 13.6849 16.2863 13.2199C16.1356 10.7431 15.3014 8.35653 13.8765 6.32511C13.7408 6.13482 13.5636 5.97787 13.3583 5.86613C13.153 5.75439 12.925 5.69078 12.6915 5.68011H10.6875M10.6875 5.68011V10.6879" stroke-linecap="round" stroke-linejoin="round"/>
															</svg>
														</div>
														<span class="tw-text-neutral-1 tw-px-2 tw-font-bold"> {{ selectedShipping.expected_days }} {{ $t("web._WORK_DAYS") }} </span>
													</div>
												</div>
											</div>

										</div>
										<div id="panelsStayOpen-collapseThree" class="accordion-collapse collapse" data-bs-parent="#accordionPanelsStayOpenExample">
											<div class="accordion-body">

												<div v-if="shippingIsLoading" class="tw-flex tw-flex-wrap">
													<div v-for="index in 3" class="lg:tw-w-1/3 sm:tw-w-1/2 tw-w-full tw-px-2">
														<div class="placeholder-glow">
															<div class="placeholder w-100" style="height: 200px"></div>
														</div>
													</div>
												</div>
												<div v-else class="">
													<!-- grouped -->
													<div v-for="(group, index) in groupedShippersArray" v-if="group.categorySlug != 'ungrouped'" :key="index" class="payment-group tw-flex tw-flex-col">
														<div class="tw-flex tw-items-center tw-gap-3 tw-h-[42px]">
															<input @input="() => (selectedShipperGroup = group.categorySlug)" type="radio" :id="`shipper-group-` + group.categorySlug" :ref="`shipper-group-` + group.categorySlug" name="shipper-group-1">
															<label :for="`shipper-group-` + group.categorySlug" class="tw-text-header tw-text-lg tw-mb-0">{{ group.categorySlug }}</label>
														</div>
														<div v-if="selectedShipperGroup == group.categorySlug">
															<!-- Shippers under the Group -->
															<div class="tw-bg-muted-light tw-px-3 tw-mt-4 tw-pt-6 tw-flex tw-flex-wrap">
																<label
																	v-for="(item, index) in group.shippingOptions"
																	:key="index"
																	v-if="(parseInt($store.state.storeCurrencyTypeId) === 2) || (parseInt($store.state.storeCurrencyTypeId) === 1 && item.id != 1 && item.id != 2)"
																	class="tw-text-dark lg:tw-w-1/3 sm:tw-w-1/2 tw-w-full tw-px-3 tw-mb-6 tw-h-auto tw-relative"
																>
																	<div class="tw-flex tw-flex-col tw-gap-4">
																		<div :class="selectedShipping == item && 'tw-border-success'" class="tw-relative tw-px-4 tw-py-3 tw-bg-light tw-border-gray tw-border d-flex tw-flex-col tw-justify-start tw-items-start tw-gap-1 tw-inline-flex">
																			<svg v-if="selectedShipping == item" :class="isRTL()?'tw-absolute tw-top-0 tw-left-0 -tw-translate-x-1/2 -tw-translate-y-1/2' :'tw-absolute tw-top-0 tw-right-0 tw-translate-x-1/2 -tw-translate-y-1/2'"  width="30" height="30" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
																				<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15Z" fill="white"/>
																				<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15ZM19.5125 12.7325C19.5875 12.6326 19.6418 12.5186 19.6721 12.3974C19.7025 12.2762 19.7083 12.1502 19.6892 12.0267C19.6701 11.9032 19.6266 11.7848 19.5611 11.6784C19.4956 11.572 19.4095 11.4797 19.3078 11.4071C19.2062 11.3344 19.091 11.2828 18.9691 11.2553C18.8473 11.2278 18.7211 11.2249 18.5981 11.2468C18.4751 11.2688 18.3577 11.3151 18.2528 11.383C18.148 11.451 18.0578 11.5392 17.9875 11.6425L13.9425 17.305L11.9125 15.275C11.7348 15.1094 11.4997 15.0192 11.2568 15.0235C11.014 15.0278 10.7822 15.1262 10.6105 15.298C10.4387 15.4697 10.3403 15.7015 10.336 15.9443C10.3317 16.1872 10.4219 16.4223 10.5875 16.6L13.4 19.4125C13.4962 19.5087 13.6123 19.5827 13.74 19.6296C13.8677 19.6764 14.0041 19.6948 14.1397 19.6837C14.2753 19.6725 14.4068 19.6319 14.5252 19.5648C14.6435 19.4977 14.7458 19.4056 14.825 19.295L19.5125 12.7325Z" fill="#25B77E"/>
																			</svg>
																			<div class="tw-text-base">{{ (item.content_translations[langsConvert[$i18n.locale]])? item.content_translations[langsConvert[$i18n.locale]].title : item.content_translations['eng'].title }}</div>
																			<span
																				class="tw-text-neutral-2 tw-whitespace-normal"
																				style="direction: rtl"
																				v-if="item.content_translations[langsConvert[$i18n.locale]] || item.content_translations['eng']"
																				v-html="(item.content_translations[langsConvert[$i18n.locale]])?item.content_translations[langsConvert[$i18n.locale]].description : item.content_translations['eng'].description"
																			></span>
																		</div>
																	</div>
																	<input @input="() => (selectedShipping = item , MoveToNext(3))" type="radio" :id="`shipper-` + item.id"
																			:ref="`shipper-` + item.id" name="shipper-group-1" :disabled="item.disabled"
																			:checked="selectedShipping == item" hidden />
																</label>
															</div>
														</div>
														<hr class="tw-border-b tw-border-gray/80 tw-my-3"/>
													</div>

													<!-- singal -->
													<div v-for="(group, index) in groupedShippersArray"  v-if="group.categorySlug == 'ungrouped'" :key="index" class="shipper-group">
														<!-- Shipping under the Group -->
														<div v-for="(item, index) in group.shippingOptions" :key="index" v-if="(parseInt($store.state.storeCurrencyTypeId) === 2) || (parseInt($store.state.storeCurrencyTypeId) === 1 && item.id != 1 && item.id != 2)" >
															<div class="tw-flex tw-items-center tw-gap-3 tw-justify-between ">
																<div class="tw-flex tw-items-center tw-gap-3 tw-min-h-[42px] tw-w-full">
																	<label :for="`shipper-group-` + item.id" class="tw-flex md:tw-flex-row tw-flex-col tw-gap-3 tw-justify-between tw-w-full tw-mb-0">
																		<div class="tw-flex tw-items-center tw-gap-3">
																			<input
																				@input="() => (selectedShipping = item , selectedShipperGroup = '' , MoveToNext(3))"
																				type="radio"
																				:id="`shipper-group-` + item.id"
																				:ref="`shipper-group-` + item.id"
																				name="shipper-group-1"
																				:disabled="item.disabled"
																				:checked="selectedShipping == item"
																			/>
																			<div class="tw-flex tw-items-center tw-gap-2 tw-mb-0">
																				<div class="tw-text-header tw-text-lg">{{ (item.content_translations[langsConvert[$i18n.locale]])? item.content_translations[langsConvert[$i18n.locale]].title : item.content_translations['eng'].title  }}</div>
																				<div class="tw-font-regular tw-text-neutral-2 tw-text-base">
																					<span class="tw-custom-badge-success">
																						<span class="" v-if="parseInt(item.shipping_fees_calculation_type) === 1 && item.shipping_fees_value > 0 ">
																							% {{ item.shipping_fees_value }}
																						</span>
																						<span  class="" v-else-if="parseInt(item.shipping_fees_calculation_type) === 2 && item.shipping_fees_value_in_exchange_rate > 0">
																							{{numberFormat(item.shipping_fees_value_in_exchange_rate, $store.state.currentPriceCurrency) }}
																						</span>
																						<span class="" v-else-if="parseInt(item.shipping_fees_calculation_type) === 3 && item.shipping_weight_fees_value_in_exchange_rate > 0">
																							{{numberFormat(item.shipping_weight_fees_value_in_exchange_rate, $store.state.currentPriceCurrency)}}
																						</span>
																						<span v-else class="">
																						{{ $t("web._Free") }}
																						</span>
																					</span>
																				</div>
																			</div>
																		</div>

																		<div v-if="item.expected_days > 0" class="tw-text-sm tw-w-fit">
																			<!-- <span class="tw-text-neutral-2"> {{ $t("web._EXPECTED_SHIPPING_DAYS") }}:</span>
																			<span class="tw-text-neutral-1 tw-font-bold"> {{ $t("web._WORK_DAYS") }} {{ item.expected_days }} </span> -->
																			<div :class="{'tw-border-primary': selectedShipping == item, 'tw-border-gray/80': selectedShipping != item}" class="tw-flex tw-items-center tw-border ">
																				<div class="tw-p-1 tw-bg-muted-light">
																					<svg width="32" height="32" viewBox="0 0 18 18" :class="{'tw-stroke-primary': selectedShipping == item, 'tw-stroke-neutral-2': selectedShipping != item}" fill="none" xmlns="http://www.w3.org/2000/svg">
																						<path d="M6.18751 14.0629C6.18751 14.3612 6.06899 14.6474 5.85801 14.8584C5.64703 15.0693 5.36088 15.1879 5.06251 15.1879C4.76415 15.1879 4.478 15.0693 4.26702 14.8584C4.05604 14.6474 3.93751 14.3612 3.93751 14.0629M6.18751 14.0629C6.18751 13.7645 6.06899 13.4783 5.85801 13.2674C5.64703 13.0564 5.36088 12.9379 5.06251 12.9379C4.76415 12.9379 4.478 13.0564 4.26702 13.2674C4.05604 13.4783 3.93751 13.7645 3.93751 14.0629M6.18751 14.0629H10.6875M3.93751 14.0629H2.53126C2.30749 14.0629 2.09288 13.974 1.93464 13.8157C1.77641 13.6575 1.68751 13.4429 1.68751 13.2191V10.6879M10.6875 14.0629H12.375M10.6875 14.0629V10.6879M1.68751 10.6879V4.96161C1.68632 4.75624 1.76136 4.55772 1.89811 4.40449C2.03485 4.25126 2.22358 4.1542 2.42776 4.13211C4.92755 3.87263 7.44748 3.87263 9.94726 4.13211C10.371 4.17561 10.6875 4.53561 10.6875 4.96161V5.68011M1.68751 10.6879H10.6875M14.625 14.0629C14.625 14.3612 14.5065 14.6474 14.2955 14.8584C14.0845 15.0693 13.7984 15.1879 13.5 15.1879C13.2016 15.1879 12.9155 15.0693 12.7045 14.8584C12.4935 14.6474 12.375 14.3612 12.375 14.0629M14.625 14.0629C14.625 13.7645 14.5065 13.4783 14.2955 13.2674C14.0845 13.0564 13.7984 12.9379 13.5 12.9379C13.2016 12.9379 12.9155 13.0564 12.7045 13.2674C12.4935 13.4783 12.375 13.7645 12.375 14.0629M14.625 14.0629H15.4688C15.9345 14.0629 16.3155 13.6849 16.2863 13.2199C16.1356 10.7431 15.3014 8.35653 13.8765 6.32511C13.7408 6.13482 13.5636 5.97787 13.3583 5.86613C13.153 5.75439 12.925 5.69078 12.6915 5.68011H10.6875M10.6875 5.68011V10.6879" stroke-linecap="round" stroke-linejoin="round"/>
																					</svg>
																				</div>
																				<span class="tw-text-neutral-1 tw-px-2 tw-font-bold"> {{ item.expected_days }} {{ $t("web._WORK_DAYS") }} </span>
																			</div>

																		</div>

																	</label>
																</div>

															</div>
															<hr class="tw-border-b tw-border-gray/80 tw-my-3"/>
														</div>
													</div>

													<!-- <div class="tw-flex tw-justify-center text-center sm:tw-relative tw-fixed tw-bottom-0 tw-left-0 tw-right-0 sm:tw-bg-[#00000000] tw-bg-light sm:tw-py-0 tw-py-3 tw-z-20 sm:tw-border-t-0 tw-border-t-2 tw-border-gray/60 tw-px-3">
														<button type="button" :disabled="!selectedShipping" class="tw-btn sm:tw-w-52 tw-w-full" @click="MoveToNext(3)">
															{{ $t('web._NEXT') }}
														</button>
													</div> -->
												</div>
											</div>
										</div>
									</div>
									<!-- End Shipping Accordion-->

									<!-- checkbox terms & condition -->
									<!-- <div v-if="contractTemplete !== null" class="col-sm-12">
										<div class="mb-3">
											<div class="form-check d-flex tw-items-center ">
												<div>
													<input class="form-check-input" type="checkbox"  style="width: 16px;height: 16px;" v-model="read_terms_and_condidtion">
												</div>
												<label class="form-check-label tw-px-2">
													{{ $t("web._IAgree1") }} <span class="tw-text-primary" @click="openTermsAndCondition()">{{ $t("web._TERMS_AND_CONDITIONS") }}</span> {{ $t("web._IAgree2") }}
												</label>
											</div>
										</div>
									</div> -->
									<!-- chat box -->
									<div id="chat-box" v-if="chatBoxNum" :class="unqualifiedProducts ? 'alert alert-danger' : 'alert alert-success'" class="tw-px-3 tw-py-4 sm:tw-block tw-hidden">

										<!-- <div class="tw-flex tw-items-center tw-gap-2 tw-mb-3">
											<svg v-if="!$store.getters.cartIsEmpty" width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
												<path d="M9 12.75L11.25 15L15 9.75M21 12C21 13.268 20.37 14.39 19.407 15.068C19.5108 15.6608 19.4701 16.2698 19.2886 16.8436C19.107 17.4173 18.7899 17.9388 18.364 18.364C17.9388 18.7899 17.4173 19.107 16.8436 19.2886C16.2698 19.4701 15.6608 19.5108 15.068 19.407C14.7222 19.8995 14.2629 20.3014 13.7288 20.5787C13.1948 20.856 12.6017 21.0005 12 21C10.732 21 9.61 20.37 8.932 19.407C8.33923 19.5107 7.73021 19.47 7.15649 19.2885C6.58276 19.1069 6.06122 18.7898 5.636 18.364C5.21013 17.9388 4.89298 17.4173 4.71142 16.8436C4.52987 16.2698 4.48925 15.6608 4.593 15.068C4.10052 14.7222 3.69862 14.2629 3.42133 13.7288C3.14403 13.1948 2.99951 12.6017 3 12C3 10.732 3.63 9.61 4.593 8.932C4.48925 8.33923 4.52987 7.73019 4.71142 7.15645C4.89298 6.58271 5.21013 6.06117 5.636 5.636C6.06122 5.21019 6.58276 4.8931 7.15649 4.71154C7.73021 4.52999 8.33923 4.48933 8.932 4.593C9.27785 4.10058 9.73722 3.69873 10.2713 3.42144C10.8053 3.14415 11.3983 2.99959 12 3C13.268 3 14.39 3.63 15.068 4.593C15.6608 4.48933 16.2698 4.52999 16.8435 4.71154C17.4172 4.8931 17.9388 5.21019 18.364 5.636C18.7898 6.06122 19.1069 6.58276 19.2885 7.15649C19.47 7.73021 19.5107 8.33923 19.407 8.932C19.8995 9.27779 20.3014 9.73715 20.5787 10.2712C20.856 10.8052 21.0005 11.3983 21 12Z" stroke="#25B77E" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
											</svg>
											<svg v-else width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
												<path d="M15.068 8.932L8.932 15.068M8.932 8.932L15.068 15.068M21 12C21 13.268 20.37 14.39 19.407 15.068C19.5108 15.6608 19.4701 16.2698 19.2886 16.8436C19.107 17.4173 18.7899 17.9388 18.364 18.364C17.9388 18.7899 17.4173 19.107 16.8436 19.2886C16.2698 19.4701 15.6608 19.5108 15.068 19.407C14.7222 19.8995 14.2629 20.3014 13.7288 20.5787C13.1948 20.856 12.6017 21.0005 12 21C10.732 21 9.61 20.37 8.932 19.407C8.33923 19.5107 7.73021 19.47 7.15649 19.2885C6.58276 19.1069 6.06122 18.7898 5.636 18.364C5.21013 17.9388 4.89298 17.4173 4.71142 16.8436C4.52987 16.2698 4.48925 15.6608 4.593 15.068C4.10052 14.7222 3.69862 14.2629 3.42133 13.7288C3.14403 13.1948 2.99951 12.6017 3 12C3 10.732 3.63 9.61 4.593 8.932C4.48925 8.33923 4.52987 7.73019 4.71142 7.15645C4.89298 6.58271 5.21013 6.06117 5.636 5.636C6.06122 5.21019 6.58276 4.8931 7.15649 4.71154C7.73021 4.52999 8.33923 4.48933 8.932 4.593C9.27784 4.10058 9.73722 3.69873 10.2713 3.42144C10.8053 3.14415 11.3983 2.99959 12 3C13.268 3 14.39 3.63 15.068 4.593C15.6608 4.48933 16.2698 4.52999 16.8435 4.71154C17.4172 4.8931 17.9388 5.21019 18.364 5.636C18.7898 6.06122 19.1069 6.58276 19.2885 7.15649C19.47 7.73021 19.5107 8.33923 19.407 8.932C19.8995 9.27779 20.3014 9.73715 20.5787 10.2712C20.856 10.8052 21.0005 11.3983 21 12Z" stroke="#F46A6A" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
											</svg>

											<h3 class="tw-text-lg tw-mb-0">
												{{ $t("web._ORDER_SUMMERY") }}
											</h3>
										</div> -->

										<ul class="tw-px-5 tw-flex tw-flex-col tw-gap-2">
											<div class="">
												<span v-html="visibleChatBoxMessage"></span>
												<span class="cursor" v-show="showCursor">&#9646;</span>
											</div>
										</ul>
									</div>
									<!-- End chat box -->
								</div>
							</div>
							<!--  End Checkout step -->

						</div>

						<!-- Side Order Summery -->
						<div class="col-lg-3 col-12">
							<div class="tw-sticky tw-top-36 summary-card mt-lg-0 tw-z-20">
								<div class="d-flex flex-column  tw-px-3 tw-py-4 tw-mt-4 tw-bg-muted-light tw-border tw-border-gray/60">
									<!-- Summery cart-->
									<div class="tw-my-2">
										<div class="d-flex justify-content-between w-100">
											<span class="tw-text-header tw-font-black tw-text-base ">
												{{ $t("web._SHOPPING_CART") }}
												<!-- ({{Object.keys($store.state.cart).length}}) -->
											</span>
											<span class="tw-text-header tw-text-base">
												<!-- change here for Offer -->
												({{Object.values($store.state.cart).reduce((a, b) => a + parseInt(b.qty), 0) + Object.values($store.state.offerCart).reduce((a, b) => a + parseInt(b.qty), 0) }}) {{ $t("web._ITEMS") }}
												<!-- {{ numberFormat(calculateSubTotalTotal(), $store.state.currentPriceCurrency) }} -->
											</span>
										</div>

										<!-- change here for Offer -->
										<div class="d-flex tw-flex-wrap tw-mx-0.5 tw-mt-3" >
											<div  v-for="(product, key) in $store.state.cart" :key="key" class="tw-p-0.5 tw-w-[25%]">
												<div class="tw-relative">
													<div class="tw-absolute tw-left-[1px] tw-bottom-[1px] tw-p-[1px] tw-text-[12px] tw-w-5 tw-h-5 tw-text-neutral-1 tw-flex tw-items-center tw-justify-center tw-bg-gray/60">
														{{ product.qty }}
													</div>
													<img class="tw-w-full tw-bg-light tw-border tw-border-gray/60" :src="product.options.image_url" alt="" />
												</div>
											</div>
											<!-- // Offer Cart -->
											 <template v-for="(group, groupId , index2) in groupedProducts">
												<div  v-for="(product, index) in group" :key="index" class="tw-p-0.5 tw-w-[25%]">
													<div class="tw-relative">
														<div class="tw-absolute tw-left-[1px] tw-bottom-[1px] tw-p-[1px] tw-text-[12px] tw-w-5 tw-h-5 tw-text-neutral-1 tw-flex tw-items-center tw-justify-center tw-bg-gray/60">
															{{ product.qty }}
														</div>
														<img class="tw-w-full tw-bg-light tw-border tw-border-primary tw-border-gray/60" :src="product.options.image_url" alt="" />
													</div>
												</div>
											 </template>

										</div>

									</div>
									<hr class="my-2" />
									<span class="tw-text-sm tw-mb-1 tw-font-black tw-mt-2">
										{{ $t("web._SUMMERY") }}
									</span>
									<!-- subTotal -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12 tw-mb-4">
										<div class="summary-labels tw-text-neutral-2">{{ $t("web._SUBTOTAL") }}</div>
										<div class="">
											<span class="tw-text-header tw-text-base">
												{{ numberFormat(calculateSubTotalTotal(), $store.state.currentPriceCurrency) }}
											</span>
										</div>
									</div>
									<!-- payment fees -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12">
										<div class="summary-labels tw-text-neutral-2">{{ $t("web._PAYMENT_FEES") }}</div>
										<div class="tw-text-header tw-text-base ">
											{{ numberFormat(calculateSelectedPaymentFees(), $store.state.currentPriceCurrency) }}
										</div>
									</div>
									<!-- Shipping fees -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12 ">
										<div class="summary-labels tw-text-neutral-2">{{ $t("web._SHIPPING_FEES") }}</div>
										<div class="tw-text-header  tw-text-base " dir="ltr">
											{{ numberFormat(calculateShippingFees(), $store.state.currentPriceCurrency) }}
											<span class="text-danger px-1"
												v-if="selectedShipping && calculateShippingFees() == 0 && authUser.country_id == 105"
												style="text-decoration: line-through;">{{ numberFormatNoCurrency(50 * exchangeRate) }}</span>
											<!-- {{calculateShippingFees()}} -->
										</div>
									</div>

									<!-- Extra fees -->
									<div v-for="(extraFee, key) in ExtraFeesOptions" :key="key"  class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12 ">
										<div class="summary-labels tw-text-neutral-2">{{extraFee.title}}</div>
										<div class="tw-text-header  tw-text-base " dir="ltr" :class="calculateExtraFees(extraFee) < 0 ?'text-danger' :''">
											{{ numberFormat(Math.abs(calculateExtraFees(extraFee)), $store.state.currentPriceCurrency) }}{{calculateExtraFees(extraFee) < 0 ? '-' : ''}}
										</div>
									</div>

									<!-- Offer Discount -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12"
										v-if="!$store.getters.offerCartIsEmpty &&  calculateTotalMyOfferDiscount() != 0">
										<div class="summary-labels tw-text-neutral-2">{{ $t("web._OFFER_DISCOUNT") }}</div>
										<div class="tw-text-header tw-text-base   text-danger">
											-{{ numberFormat(calculateTotalMyOfferDiscount(), $store.state.currentPriceCurrency)
											}}
										</div>
									</div>

									<!-- Discount -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12"
										v-if="(!ShowRetailAndDistrebuteStore || parseInt($store.state.shoppingType) === 2) && parseInt(this.$store.state.storeCurrencyTypeId) === 2 && calculateRetailDiscountFees() != 0">
										<div class="summary-labels tw-text-neutral-2">{{  (ShowRetailAndDistrebuteStore) ? $t("web._retail_discount") : $t("web._shopping_discount")  }}</div>
										<div class="tw-text-header tw-text-base   text-danger">
											-{{ numberFormat(calculateRetailDiscountFees(), $store.state.currentPriceCurrency)
											}}
										</div>
									</div>
									<!--								v-if="couponCodeDiscount > 0"-->
									<!-- Coupon Discount -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12 "
										v-if="calculateCouponDiscount() > 0">
										<div class="summary-labels tw-text-neutral-2">{{ $t("web._COUPON_DISCOUNT") }}</div>
										<div class="tw-text-header   tw-text-base text-danger">
											-{{ numberFormat(calculateCouponDiscount(), $store.state.currentPriceCurrency) }}
										</div>
									</div>

									<!-- Referral Discount -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12 "
										v-if="referralCodeDiscount > 0">
										<div class="summary-labels tw-text-neutral-2">{{ $t("web._REFERRAL_CODE_DISCOUNT") }}</div>
										<div class="tw-text-header   tw-text-base text-danger">
											-{{ numberFormat(calculateReferralDiscount(), $store.state.currentPriceCurrency) }}
										</div>
									</div>
									<!-- Total -->
									<div class="d-flex justify-content-between w-100 tw-my-1 col-lg-2 col-md-4 col-12 tw-text-lg tw-mt-3">
										<div class="tw-text-neutral-2">
											{{ $t("web._TOTAL") }}
										</div>
										<div class="">
											<span class="tw-text-header tw-font-bold" v-text="numberFormat(calculateGrandTotal(), $store.state.currentPriceCurrency)">
											</span>
										</div>
									</div>

									<hr class="my-2" />

									<!-- OSV code -->
									<div
										v-if="$store.state.storeCurrencyTypeId && parseInt($store.state.storeCurrencyTypeId) === 2"
										class="coupon-input d-flex flex-column gap-1 col-lg-2 col-md-4 col-12 tw-mt-2"
										style="width: 100%"
									>
										<!-- <a
											role="button"
											@click.prevent="showCouponInput"
											class="tw-text-primary coupon-input text-nowrap text-decoration-underline">
											{{ $t('web._DO_YOU_HAVE_COUPON_CODE') }}
										</a> -->
										<button @click.prevent="showCouponInput" class="tw-flex tw-justify-between tw-items-center tw-border tw-border-gray tw-bg-light tw-p-2" >
											<div class="tw-flex tw-items-center tw-gap-2">
												<svg width="24" height="24" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
													<path d="M11 4V4.5M11 6.5V7M11 9V9.5M11 11.5V12M5 8.5H8.5M5 10H7M2.25 3.5C1.836 3.5 1.5 3.836 1.5 4.25V6.26733C1.8045 6.44265 2.05742 6.69512 2.23328 6.9993C2.40914 7.30348 2.50174 7.64864 2.50174 8C2.50174 8.35136 2.40914 8.69652 2.23328 9.0007C2.05742 9.30488 1.8045 9.55735 1.5 9.73267V11.75C1.5 12.164 1.836 12.5 2.25 12.5H13.75C14.164 12.5 14.5 12.164 14.5 11.75V9.73267C14.1955 9.55735 13.9426 9.30488 13.7667 9.0007C13.5909 8.69652 13.4983 8.35136 13.4983 8C13.4983 7.64864 13.5909 7.30348 13.7667 6.9993C13.9426 6.69512 14.1955 6.44265 14.5 6.26733V4.25C14.5 3.836 14.164 3.5 13.75 3.5H2.25Z" stroke="#006AB4" stroke-linecap="round" stroke-linejoin="round"/>
												</svg>
												<span class="tw-text-neutral-2 ">{{ $t('web._APPLY_OSV_VOUCHER') }}</span>
											</div>

											<svg :class="isRTL() && 'tw-rotate-180'" class="tw-transition-all tw-stroke-gray" width="16" height="16" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
												<path d="M4.125 2.25L7.875 6L4.125 9.75" stroke-linecap="round" stroke-linejoin="round"/>
											</svg>

										</button>

									</div>

									<!-- Referral code -->
									<div
										v-if="$store.state.storeCurrencyTypeId
										&& parseInt($store.state.storeCurrencyTypeId) === 2
										&& (!ShowRetailAndDistrebuteStore || parseInt($store.state.shoppingType) === 1)"
										class="referral-input d-flex flex-column gap-1 w-100 tw-my-2 col-lg-2 col-md-4 col-12"
									>
										<!-- <a role="button" @click.prevent="showReferralInput"
											class="tw-text-primary referral-input text-nowrap text-decoration-underline">{{
												$t('web._DO_YOU_HAVE_REFERRAL_CODE') }}</a> -->
										<div class="tw-flex tw-flex-col gap-1">
											<!-- <input
												@input="editReferralCode()"
												v-model="referralCode"
												type="text"
												v-click-outside="{ handler: hideReferralInput, exclude: ['referral-input'] }"
												class="referral-input form-control"
												:placeholder="$t('web._ENTER_YOU_REFERRAL_CODE')"
											/> -->


											<div class="ref-input-parent tw-relative tw-flex tw-items-center tw-gap-2 tw-border tw-border-gray tw-bg-muted-light tw-p-2" >
												<svg class="tw-absolute " :class="isRTL() ? 'tw-right-2' : 'tw-left-2'" width="24" height="24" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
													<path d="M9.99969 12.752C10.5683 12.9171 11.1576 13.0006 11.7497 13C12.702 13.0014 13.6418 12.7843 14.497 12.3653C14.5223 11.7678 14.3521 11.1783 14.0122 10.6862C13.6723 10.1941 13.1813 9.82624 12.6135 9.63834C12.0457 9.45043 11.4321 9.45275 10.8658 9.64493C10.2995 9.83711 9.81119 10.2087 9.47502 10.7033M9.99969 12.752V12.75C9.99969 12.008 9.80902 11.31 9.47502 10.7033M9.99969 12.752V12.8227C8.71668 13.5954 7.24676 14.0025 5.74902 14C4.19502 14 2.74102 13.57 1.49969 12.8227L1.49902 12.75C1.49851 11.8063 1.8121 10.8893 2.39035 10.1435C2.96859 9.39773 3.77861 8.86562 4.69268 8.63107C5.60676 8.39651 6.57291 8.47285 7.4388 8.84806C8.30469 9.22327 9.02108 9.87599 9.47502 10.7033M7.99969 4.25C7.99969 4.84674 7.76264 5.41903 7.34068 5.84099C6.91872 6.26295 6.34643 6.5 5.74969 6.5C5.15295 6.5 4.58066 6.26295 4.1587 5.84099C3.73674 5.41903 3.49969 4.84674 3.49969 4.25C3.49969 3.65326 3.73674 3.08097 4.1587 2.65901C4.58066 2.23705 5.15295 2 5.74969 2C6.34643 2 6.91872 2.23705 7.34068 2.65901C7.76264 3.08097 7.99969 3.65326 7.99969 4.25ZM13.4997 5.75C13.4997 6.21413 13.3153 6.65925 12.9871 6.98744C12.6589 7.31563 12.2138 7.5 11.7497 7.5C11.2856 7.5 10.8404 7.31563 10.5123 6.98744C10.1841 6.65925 9.99969 6.21413 9.99969 5.75C9.99969 5.28587 10.1841 4.84075 10.5123 4.51256C10.8404 4.18437 11.2856 4 11.7497 4C12.2138 4 12.6589 4.18437 12.9871 4.51256C13.3153 4.84075 13.4997 5.28587 13.4997 5.75Z" stroke="#006AB4" stroke-linecap="round" stroke-linejoin="round"/>
												</svg>

												<input
													@input="editReferralCode()"
													v-model="referralCode"
													maxlength="6"
													type="text"
													class="tw-text-neutral-2 referral-input form-control tw-border-none tw-p-0 tw-bg-[#00000000]"

													:placeholder="$t('web._ENTER_YOU_REFERRAL_CODE')"
													v-click-outside="{ handler: hideReferralInput, exclude: ['referral-input'] }"
												/>
											</div>

											<div v-if="isCouponReferralLoadingMode" class="text-center tw-text-success p-3">
												<div>
													<span class="spinner-grow spinner-grow-sm"></span>
												</div>
												<div>
													 is loading ..
												</div>
											</div>
											<div v-else-if="referralCode.length == 6" class="tw-text-base tw-mt-2">
												<span v-if="referralCodeIsValid">
													<span class="tw-text-neutral-2">{{ $t("web._REFERRAL_CODE") }}:</span><br>
													<span class="tw-font-bold tw-text-success">{{ referralCodeOwnerEmail }}</span>
												</span>
												<span class="tw-flex tw-gap-1 tw-items-center" v-else>
													<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="tw-w-5 tw-h-5 tw-stroke-danger">
														<path stroke-linecap="round" stroke-linejoin="round" d="m9.75 9.75 4.5 4.5m0-4.5-4.5 4.5M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
													</svg>

													<span class="tw-text-danger">{{ $t("web._INVALID_REFERRAL_CODE") }}</span>
												</span>
											</div>
										</div>
									</div>
									<hr class="my-2" />
									<!-- BV  Card-->
									<div
										class="tw-px-3 tw-py-1 tw-bg-light tw-border tw-border-gray/80 tw-justify-start tw-items-center tw-gap-2 tw-flex tw-my-2"
										v-if="(!ShowRetailAndDistrebuteStore || parseInt($store.state.shoppingType) === 1) && $store.state.storeCurrencyTypeId
											&& parseInt($store.state.storeCurrencyTypeId) === 2
											&& $store.state.authUser
											&& $store.state.authUser.properties
											&& $store.state.authUser.properties.package_id >= 1
											&& $store.state.authUser.properties.referrer_id != null
											&& $store.state.storeCurrencyTypeId
											&& parseInt($store.state.storeCurrencyTypeId) === 2 &&
											calculateBVTotal() > 0">
										<div class="flex-shrink-0">
											<img  class="tw-w-10 tw-h-10 tw-aspect-square" src="/images/BV-up.svg" alt="">
										</div>
										<div class="">
											<div class="tw-text-success tw-text-base c-font-black" v-if="calculateBVTotal()">
												+ {{ calculateBVTotal() }} BV
											</div>
											<!-- TODO: translate -->
										</div>
									</div>

									 <div  v-if="contractTemplete !== null" class="tw-py-1">
										<div class="d-flex  tw-items-start ">
											<div>
												<input class="form-check-input" type="checkbox"  style="width: 14px;height: 14px;" v-model="read_terms_and_condidtion">
											</div>
											<label class="tw-px-2">
												{{ $t("web._IAgree1") }} <span class="tw-text-primary" @click="openTermsAndCondition()">{{ $t("web._TERMS_AND_CONDITIONS") }}</span> {{ $t("web._IAgree2") }}
											</label>
										</div>
									</div>
									<!-- submit button -->
									<div class="tw-flex tw-flex-col tw-fixed tw-bg-light tw-justify-center sm:tw-relative tw-bottom-0 tw-left-0 tw-right-0 sm:tw-bg-[#00000000] sm:tw-py-0 tw-py-3 tw-z-20 sm:tw-border-t-0 tw-px-3 sm:tw-px-0 tw-border-t-2 tw-border-gray/60">
										<div id="chat-box" v-if="chatBoxNum" class="tw-px-3 tw-py-4 alert alert-success sm:tw-hidden tw-block">
											<ul class="tw-px-5 tw-flex tw-flex-col tw-gap-2">
												<div class="">
													<span v-html="visibleChatBoxMessage"></span>
													<span class="cursor" v-show="showCursor">&#9646;</span>
												</div>
											</ul>
										</div>

										<a
											role="button"
											class="tw-btn tw-py-2 tw-gap-1 tw-px-12 tw-mt-3 hover:tw-text-dark tw-w-full text-center"
											:class="{ 'disabled': (!checkPlaceOrder) }"
											@click="submitForm()"
										>
											<i v-if="generateOrderLoadingMode" class="bx bx-loader bx-spin font-size-16 align-middle me-2"></i>
											<!-- <span v-if="chatBoxNum == 4">
												<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 30 30" fill="none">
													<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15Z" fill="#424050"></path>
													<path fill-rule="evenodd" clip-rule="evenodd" d="M2.8125 15C2.8125 8.26875 8.26875 2.8125 15 2.8125C21.7313 2.8125 27.1875 8.26875 27.1875 15C27.1875 21.7313 21.7313 27.1875 15 27.1875C8.26875 27.1875 2.8125 21.7313 2.8125 15ZM19.5125 12.7325C19.5875 12.6326 19.6418 12.5186 19.6721 12.3974C19.7025 12.2762 19.7083 12.1502 19.6892 12.0267C19.6701 11.9032 19.6266 11.7848 19.5611 11.6784C19.4956 11.572 19.4095 11.4797 19.3078 11.4071C19.2062 11.3344 19.091 11.2828 18.9691 11.2553C18.8473 11.2278 18.7211 11.2249 18.5981 11.2468C18.4751 11.2688 18.3577 11.3151 18.2528 11.383C18.148 11.451 18.0578 11.5392 17.9875 11.6425L13.9425 17.305L11.9125 15.275C11.7348 15.1094 11.4997 15.0192 11.2568 15.0235C11.014 15.0278 10.7822 15.1262 10.6105 15.298C10.4387 15.4697 10.3403 15.7015 10.336 15.9443C10.3317 16.1872 10.4219 16.4223 10.5875 16.6L13.4 19.4125C13.4962 19.5087 13.6123 19.5827 13.74 19.6296C13.8677 19.6764 14.0041 19.6948 14.1397 19.6837C14.2753 19.6725 14.4068 19.6319 14.5252 19.5648C14.6435 19.4977 14.7458 19.4056 14.825 19.295L19.5125 12.7325Z" fill="#fff"></path>
												</svg>
											</span> -->

											<span>{{ $t("web._PLACE_ORDER") }}</span>
											<!-- v-if="chatBoxNum != 4" -->
											<span >
												({{ `${Math.min(Math.max(chatBoxNum - 1, 0), 3) }/3`
												}})
											</span>
										</a>
									</div>
								</div>
							</div>

						</div>
					</div>
				</div>
			</div>


		</div>

		<!-- Address Modal -->
		<div class="modal fade bs-example-modal-center bs-example-modal-lg" tabindex="-1" role="dialog" id="address-modal"
			ref="addressModal" aria-hidden="true">
			<div class="modal-dialog modal-dialog-centered modal-lg">
				<div class="modal-content">
					<div class="modal-header">
						<h5 class="modal-title">{{ $t("web._ADD_NEW_ADDRESS") }}</h5>
						<button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
					</div>
					<div class="modal-body">
						<div class="mb-3">
							<!-- Form group -->
							<div class="row">
								<!-- Address -->
								<div class="col-md-12 col-sm-12">
									<div class="mb-3">
										<label class="form-label">
											{{ $t("web._ADDRESS") }}<i class="text-danger">*</i>
										</label>
										<input type="text" class="form-control" placeholder="Address" v-model="address.unit_address" />
									</div>
								</div>
								<!-- Country -->
								<div class="col-md-6 col-sm-12">
									<div class="mb-3">
										<label class="form-label">
											{{ $t("web._COUNTRY") }}<i class="text-danger">*</i>
										</label>
										<select class="form-control" v-model="address.country_id"
											@change="fetchCountryCites(address.country_id); fetchForm(address.country_id)">
											<option disabled value="null"><em>Select Country</em></option>
											<option v-for="country in countries" v-text="country.title" :value="country.id">
											</option>
										</select>
									</div>
								</div>
								<!-- State -->
								<div class="col-md-6 col-sm-12">
									<div class="mb-3">
										<label class="form-label">
											{{ $t("web._STATE") }}<i class="text-danger">*</i>
										</label>
										<select class="form-control" v-model="address.city_id">
											<option disabled value="null"><em>{{ $t("web._SELECT_STATE") }}</em></option>
											<option v-for="city in cities" v-text="city.name" :value="city.id"></option>
										</select>
									</div>
								</div>

								<!-- Zip code -->
								<div class="col-md-6 col-sm-12">
									<div class="mb-3">
										<label class="form-label">
											{{ $t("web._ZIP_CODE") }}<i class="text-danger">*</i>
										</label>
										<input type="text" class="form-control" placeholder="" v-model="address.zip_code" />
									</div>
								</div>

								<!-- Phone number -->
								<div class="col-md-6 col-sm-12">
									<div class="mb-3">
										<label class="form-label">
											{{ $t("web._PHONE_NUMBER") }}<i class="text-danger">*</i>
										</label>
										<input type="text" class="form-control" placeholder="" v-model="address.phone_number" />
									</div>
								</div>


								<!-- Dynamic form inputs -->
								<div class="row mx-0 px-0" v-for="(row, idx) in templateForm">
									<div :class="col.widthClass" v-for="(col, idy) in row.columns">
										<div class="" v-for="(input, idz) in col.components">
											<div class="mb-3">
												<div>
													<label class="form-label" v-html="input.titles[langsConvert[$i18n.locale]]"></label>
													<i v-if="input.required" class="text-danger">*</i>
												</div>
												<!-- file -->
												<input :type="input.type" class="form-control" v-if="input.type == 'file'" :name="input.name"
													:required="input.required" @change="onFileChange">
												<input :type="input.type" class="form-control" v-if="input.cat == 'input' && input.type != 'file'"
													:required="input.required" v-model="template_form[input.name]"
													@input="handleInputChange(input.name, input.titles)">
												<textarea class="form-control" v-if="input.cat == 'textarea'" :required="input.required"
													v-model="template_form[input.name]" @input="handleInputChange(input.name, input.titles)"></textarea>
											</div>
										</div>
									</div>
								</div>

								<!-- Notes -->
								<div class="col-md-12 col-sm-12">
									<div class="mb-3">
										<label class="form-label">
											{{ $t("web._NOTE") }}<i class="text-danger">*</i>
										</label>
										<textarea class="form-control" id="" cols="30" rows="5" v-model="address.notes"></textarea>
									</div>
								</div>
								<!-- is default -->
								<div class="col-md-6 col-sm-12">
									<div class="mb-3">
										<div class="form-check">
											<input class="form-check-input" type="checkbox" id="formCheck2" v-model="address.default_status">
											<label class="form-check-label" for="formCheck2">
												{{ $t("web._MARK_AS_DEFAULT") }}
											</label>
										</div>
									</div>
								</div>
							</div>

							<div class="text-center">
								<a role="button" class="tw-btn  hover:tw-text-dark mt-3 ms-1" @click="() => createAddress()"
									v-if="!updateAction && !createAddressLoadingMode">
									{{ $t("web._SUBMIT") }}
								</a>
								<a
									role="button" class="tw-btn  hover:tw-text-dark mt-3 ms-1"
									@click="() => updateAddress()"
									v-else-if="updateAction && !createAddressLoadingMode"
								>
								{{ $t("web._SUBMIT") }}
								</a>
								<div class="text-center" v-else>
									<span class="text-success">
										<i class="bx bx-loader bx-spin font-size-16 align-middle"></i>
										{{ $t("web._PLEASE_WAIT") }} ..
									</span>
								</div>
							</div>

						</div>
					</div>
				</div>
				<!-- /.modal-content -->
			</div>
			<!-- /.modal-dialog -->
		</div>
		<!-- /.modal -->

		<!-- OSV Modal -->
		<div class="modal fade bs-example-modal-osv-center bs-example-modal-lg" tabindex="-1" role="dialog" id="osv-modal"
			ref="osvModal" aria-hidden="true">
			<div class="modal-dialog modal-dialog-centered modal-lg">
				<div class="modal-content">
					<div class="modal-header">
						<h5 class="modal-title">{{ $t('web._ADD_NEW_OSV_VOUCHER') }}</h5>
						<button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
					</div>
					<div class="modal-body">
						<div class="mb-3">
								<table class="w-100 c-table tw-mt-4">
									<tr class="tw-bg-muted-light">
										<th class="tw-text-header  tw-text-base tw-p-3">
											{{$t('web._OSV_VOUCHER')}}
										</th>
										<th class="tw-text-header  tw-text-base tw-p-3">
											{{$t('web._STATUS')}}
										</th>
										<th class="tw-text-header  tw-text-base tw-p-3">
											{{$t('web._Amount')}}
										</th>
										<th class="tw-text-header  tw-text-base tw-p-3">

										</th>
									</tr>

									<tr  v-if="osvCoupon.status !== 0"  class="tw-my-3 tw-font-regular tw-bg-white" v-for="osvCoupon in osvCoupons"  :key="key">
										<td class="tw-my-3 tw-mx-4 tw-justify-between tw-items-center tw-inline-flex">
											<input type="text" class="form-control"  :disabled="osvCoupon.status == 1"
												@input="validateOSVCode(osvCoupon)"	:placeholder="$t('web._OSV_VOUCHER')" v-model="osvCoupon.code">
										</td>
										<td class="tw-my-3">
											<span v-if="osvCoupon.status !== 0" >
												<span class="tw-text-header" v-if="osvCoupon.status === 1">  <i class="bx bx-check-circle tw-text-success tw-text-header"> </i>{{$t('web._VALID_DISCOUNT_CODE')}}</span>
												<span class="tw-text-header" v-else-if="osvCoupon.status === 2"> <i class="bx bx-no-entry tw-text-danger "> </i> {{$t('web._INVALID_DISCOUNT_CODE')}}</span>
											</span>
										</td>
										<td class="tw-my-3 tw-justify-between tw-items-center tw-inline-flex">
											<span v-if="osvCoupon.amount" class="">
													{{ numberFormat(osvCoupon.amount * exchangeRate,
														$store.state.currentPriceCurrency) }}
												</span>
											<span v-else class="tw-text-header tw-font-black tw-mx-3"  > - </span>
										</td>
										<td class="tw-my-3">
											<button
													class=" tw-text-danger tw-px-2 tw-py-1"
													@click="deleteOSVCoupon(osvCoupon)"><i class=" bx bx-trash tw-text-base"></i>
											</button>
										</td>
									</tr>
									<tr v-if="osvCouponInCheckingCode">
										<td colspan="7" class="text-center tw-text-success p-3">
											<div>
												<span class="spinner-grow spinner-grow-sm"></span>
											</div>
											<div>
												{{$t('web._YOUR_OSV_UNDER_CHECK')}}
											</div>
										</td>
									</tr>

									<td class="tw-my-3 tw-mx-4 tw-justify-between tw-items-center tw-inline-flex">
											<input type="text" class="form-control"
												@input="addNewOSVCoupon($event)"  v-model="addOsvCoupon" :placeholder="$t('web._ADD_NEW_OSV_VOUCHER')">
									</td>

									<tr class="">
										<th class="tw-text-header  tw-text-base tw-p-3">
										</th>
										<th class="tw-text-header  tw-text-base tw-p-3">
											<div class="tw-flex tw-flex-col">
												<div class="tw-flex tw-gap-5 tw-py-2">
													{{$t('web._your_discount')}}
												</div>
												<div class="tw-flex tw-gap-5 tw-py-2">
													{{$t('web._ORDER_AMOUNT')}}

												</div>
												<div class="tw-flex tw-gap-5 tw-py-2">
													{{$t('web._TOTAL')}}
												</div>
											</div>
										</th>
										<th class="tw-text-header  tw-text-base tw-p-3">
											<div class="tw-flex tw-flex-col">
												<div class="tw-py-2">
														<span class="tw-text-header tw-text-base text-danger" v-if="calculateCouponDiscount() != 0">
															{{ numberFormat(calculateCouponDiscount(),
																$store.state.currentPriceCurrency) }}-
														</span>
														<span v-else class="tw-text-header tw-font-black tw-mx-3"  > - </span>
													</div>
													<div class="tw-py-2">
														<span class="tw-text-header tw-text-base">
															{{ numberFormat(calculateGrandTotal() + calculateCouponDiscount(), $store.state.currentPriceCurrency) }}
														</span>
													</div>
													<div class="tw-py-2">
														<span class="tw-text-header tw-text-base text-success">
															{{ numberFormat(calculateGrandTotal(), $store.state.currentPriceCurrency) }}
														</span>
													</div>
											</div>
										</th>
										<th class="tw-text-header tw-text-base tw-p-3">
										</th>
									</tr>

								</table>
						</div>
					</div>
				</div>
				<!-- /.modal-content -->
			</div>
			<!-- /.modal-dialog -->
		</div>
		<!-- /.modal -->

			<!-- Term Of Use Modal -->
		<div class="modal fade bs-example-modal-terms-of-use bs-example-modal-lg" tabindex="-1" role="dialog" id="terms-of-use-modal"
			ref="osvModal" aria-hidden="true">
			<div class="modal-dialog modal-dialog-centered modal-lg">
				<div class="modal-content">
					<div class="modal-header">
						<h5 class="modal-title">{{ $t('web._TERMS_AND_CONDITIONS') }}</h5>
						<button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
					</div>
					<div class="modal-body">
						 <div v-html="contractTemplete" class="container" ></div>
					</div>
					<div class="modal-footer">
						<div class="d-flex gap-2 justify-content-center">
							<a role="button" class="tw-outline-btn" data-bs-dismiss="modal" aria-label="Close">
								{{ $t("web.CANCEL") }}
							</a>
							<a role="button" class="tw-btn" @click="agreeToTerms()" data-bs-dismiss="modal" aria-label="Close">
								{{ $t("web._IAgree") }}
							</a>
						</div>
					</div>
				</div>
				<!-- /.modal-content -->
			</div>
			<!-- /.modal-dialog -->
		</div>
		<!-- /.modal -->

	</div>
</template>

<script>

export default {
	data() {
		return {
			isLoading: true,
			isLoadingGlobal: true,
			AddressisLoading:false,
			paymentMethodIsLoading:false,
			shippingIsLoading:false,
			updateAction:false,
			error: {
				thereAnError: false,
				message: '',
			},
			submitFormLoading:false,
			shoppingType: null,
			shoppingTypeCount: 0,
			stepNum:0,
			authUser: null,
			physicalProductSubTotal: 0,

			payments: {},
			groupedPaymentsArray: [], // Your grouped payments array

			groupedShippersArray:[],
			selectedPaymentGroup:'',
			selectedShipperGroup:'',
			addresses: {},
			countries: {},
			cities: {},
			selectedCity: null,
			shippingOptions: {},
			address: [
				{
					unit_address: "",
					unit_number: "",
					state: "",
					city: "",
					city_id: "",
					phone_number: "",
				},
			],
			selectedPayment: null,
			selectedAddress: null,
			selectedShipping: null,
			addressModal: null,

			order: null,
			generatePaymentLoadingMode: false,
			generateOrderLoadingMode: false,
			createAddressLoadingMode: false,

			isQualifiedForShopping: true,
			checkIrRegisteration: false,
			RegisterationProductId: null,
			onlyRegisterationProductInCart: false,
			RegisterationProduct: {},
			needIrRegisteration: false,
			hideAddressAndShipping: false ,
			addRegisterationProductLoading: false,
			errorArray: [],
			deleteAllMode: false,
			userWallet: null,
			paymentIsRestricted: false ,
			isCouponInputVisible: false,
			isCouponLoadingMode: false,
			couponCode: '',
			couponCodeDiscount: 0,
			couponCodeDiscountAmount: 0,
			couponCodeTypeId: null,
			couponCodeDiscountAmountType: 1,
			exchangeRate: 1,
			//
			osvCoupons: [
				{
					code: '',
					amount: 0,
					used: 0,
					remaining: 0,
					status: 0,
				},
			],
			addOsvCoupon: '',
			osvCouponInCheckingCode: null,
			disabledOsvPaymentMethod:true,
			isReferralInputVisible: false,
			referralCode: '',
			referralCodeDiscount: 0,
			referralCodeIsValid: false,
			referralCodeOwnerEmail: '',
			isCouponReferralLoadingMode: false,
			template_form: {},
			formTranslation: {},
			templateForm: [],

			expected_shipping_days: null,

			tickets:{
				bavo:false,
				academy:false,
			},
			checkoutCollapses: {},
			// hide_in_retail:[],
			// retail_unqualified: false,
			chatBoxNum: null,
			chatBoxMessage: this.$t("web._CURRENT_USER_SHOPPING"),
			visibleChatBoxMessage: '',
			showCursor: true,
			currentCharIndex: 0,
			ExtraFeesOptions:{},
			total_offer_bv_the_user_alreay_has:0,
			read_terms_and_condidtion:0,
			contractData:[],
			contractTemplete:null,
		};
	},
	created() {

		// check if suer is login
		if (!this.$store.state.authUser) {
			// window.location.replace("/login");
		}
		if (parseInt(this.$store.state.storeCurrencyTypeId) === 2) {
			this.fetchExtraFees();
		}

		this.fetchUserData();
		this.fetchContract();
		// this.fetchWalletAndPaymentData();
		this.fetchWalletData();
		this.fetchAddresses();
		this.hasTickets();
	},
	mounted() {

		// if (!this.$store.state.authUser){
		// 	window.location.replace("/login");
		// }
		// else if (this.$store.getters.cartIsEmpty) {
		// 	this.error.message = this.$t("web._SHOPPING_CART_IS_EMPTY") ;
		// 	this.error.thereAnError = true;
		// }

		this.addressModal = new bootstrap.Modal(this.$refs.addressModal);

		// if (parseInt(this.$store.state.storeCurrencyTypeId) === 1) {
		// 	this.setShoppingType(2)
		// }else{
		// 	this.setShoppingType(1)
		// }
		this.hasTickets();
	},

	computed:{
		TotalOsvDiscount() {

		},
		checkPlaceOrder(){
			return  (!this.$store.getters.cartIsEmpty || !this.$store.getters.offerCartIsEmpty) && this.selectedPayment && this.selectedAddress &&  (this.selectedShipping || this.hideAddressAndShipping ) && !this.unqualifiedProducts && this.read_terms_and_condidtion;
		},
		ShowRetailAndDistrebuteStore() {
			return this.$store.state.authUser && this.$store.state.authUser.properties && this.$store.state.authUser.properties.hasOwnProperty('retail_status') &&  parseInt(this.$store.state.authUser.properties.retail_status) == 1 ;
		},
		unqualifiedProducts(){
			return  this.ShowRetailAndDistrebuteStore && (this.$store.getters.productIsNotQualifiedForCheckout.length > 0 || this.$store.getters.productIsNotQualifiedForDistribute.length > 0)
		},
		groupedProducts() {
			const groups = {};
			const offerCart = this.$store.state.offerCart;

			if (typeof offerCart === 'object' && offerCart !== null) {
			const keys = Object.keys(offerCart);
			keys.forEach((key) => {
				const product = offerCart[key];
				const groupId = product.options.group_id;
				if (!groups[groupId]) {
				groups[groupId] = [];
				}
				groups[groupId].push(product);
			});
			}

			return groups;
		},
		checkUserPackage(){
			return this.$store.state.authUser && (this.$store.state.authUser.properties.package_id < 6 || this.$store.state.authUser.properties.package_id == 11) ;
		},
		calculateMyOfferTotalBvInCart() { // change here
			// calculate My Offer Cart
			const offerkeys = Object.keys(this.$store.state.offerCart);
			var offerAmount = 0;
			offerkeys.forEach((key) => {
				var offerItem = this.$store.state.offerCart[key];
				if (offerItem && this.isCartItemQualifiedForCheckout(offerItem)) {
					var offerBv = offerItem.options.sale_bv;
					if (offerItem.options.have_discount) {
						offerBv = offerItem.options.sale_bv_with_discount;
					}
					offerAmount += offerBv * offerItem.qty;
				}
			});
			return offerAmount;
		},

	},

	methods: {
		handleInputChange(name, titles) {
			this.formTranslation[name] = titles
			this.template_form.translations = this.formTranslation
		},

		onFileChange(event) {
			const file = event.target.files[0];
			const name = event.target.name;
			this.template_form[name] = file
		},

		setShoppingType(type) {
			this.$store.commit("setShoppingType", type);
		},

		fetchUserData() {
			this.isLoading = true;
			return axios // TODO: change this to use frontend api that check for status
				.get("/axios/web/get/user/data")
				.then(async (response) => {
					this.authUser = response.data.user;
					this.exchangeRate = response.data.exchange_rate;
					this.isQualifiedForShopping = response.data.is_qualified_for_shopping;
					this.checkIrRegisteration = response.data.need_ir_registeration;
					this.total_offer_bv_the_user_alreay_has = response.data.calculate_user_offer_total_bv ;
					if (this.checkIrRegisteration) {
						await this.checkRegisterProduct();
						if(this.ShowRetailAndDistrebuteStore){
							this.setShoppingType(1);
						}
					}
					this.errorArray = response.data.error_array;
					// if (this.authUser.properties.package_id < 2) {
					// 	this.setShoppingType(2);
					// }
					this.isLoading = false;
					if (!this.authUser || !this.authUser.properties) {
						window.location.replace("/login-keycloak");
					}
				})
				.catch((err) => {
					console.log('from user error')
					console.log(err)
					this.error.message = 'User Error';
					this.error.thereAnError = true;
					this.isLoading = false;
				});
		},

		fetchContract(){
			return axios // TODO: change this to use frontend api that check for status
				.get("/axios/web/get/checkout/contract/data",{
				params: {
					lang: this.$i18n.locale,
				}
			})
				.then(async (response) => {
					this.contractData = response.data.data;
					if(this.contractData){
						if(this.contractData.use_file_as_contract == 1){
						this.contractTemplete = this.contractData.file ;
						}else{
							this.contractTemplete = this.contractData.template
						}
					}else{
						this.read_terms_and_condidtion = 1;
					}
				})
				.catch((err) => {

				});
		},
		// async checkRegisterProduct() {
		// 	await this.fetchRegisterationProductId();
		// 	// check if cart contain Registeration Product to complete the proccess
		// 	await this.checkIfRegisterationProductOnlyInCart();
		// 	if (!this.onlyRegisterationProductInCart) {
		// 		// stop Him
		// 		this.needIrRegisteration = true;
		// 	} else {
		// 		this.needIrRegisteration = false;
		// 		this.hideAddressAndShipping = true ;
		// 	}
		// },

		async checkRegisterProduct(){
				await this.fetchRegisterationProductId();
				// check if cart contain Registeration Product to complete the proccess
				await this.checkIfRegisterationProductOnlyInCart();
				if(!this.onlyRegisterationProductInCart){
					// stop Him
					this.needIrRegisteration = true ;
				}else{
					this.needIrRegisteration = false ;
					// this.steps.splice(3, 1);
					// Change the index of step 5 (Summary) to 3
					// this.steps[3].index = 4;
					this.hideAddressAndShipping = true ;
				}
		},
		

		checkIfRegisterationProductOnlyInCart() { // change here
			const keys = Object.keys(this.$store.state.cart);
			let onlyRegisterationProduct = false;
			let productCount = 0;
			keys.forEach((key) => {
				var item = this.$store.state.cart[key];
				if (item.id == this.RegisterationProductId) {
					onlyRegisterationProduct = true;
				}
				productCount += item.qty;

			});
			if (productCount == 1 && onlyRegisterationProduct) {
				this.onlyRegisterationProductInCart = true;
			} else {
				this.onlyRegisterationProductInCart = false;
			}
		},

		fetchPaymentMethods() {
			this.paymentMethodIsLoading = true ;
			// this.isLoading = true;
			return axios // TODO: change this to use frontend api that check for status
				.get("/axios/order/user/payments/get/data", {
					params: {
						per_page: 15,
						search_key: this.searchKey,
						lang: this.$i18n.locale,

					},
				})
				.then((response) => {
					this.payments = response.data.items;

					this.$nextTick(() => {
						let grandTotal = this.calculateGrandTotal();
						this.$nextTick(() => {
							if (grandTotal > this.userWallet) {
								//disable payment method wallet or point
								if (this.$store.state.storeCurrencyTypeId && parseInt(this.$store.state.storeCurrencyTypeId) === 2) {
									this.payments.filter(e => e.id == 1)[0].disabled = true;
								} else {
									this.payments.filter(e => e.id == 2)[0].disabled = true;
								}
							} else {
								if (this.$store.state.storeCurrencyTypeId && parseInt(this.$store.state.storeCurrencyTypeId) === 2) {
									this.payments.filter(e => e.id == 1)[0].disabled = false;
								} else {
									this.payments.filter(e => e.id == 2)[0].disabled = false;
								}
							}
							// this.isLoading = false;
						});
					})
					// make payments as groups
					const groupedPayments = {};
					this.payments.forEach((payment) => {
						payment.payment_cards = (JSON.parse(payment.payment_cards) == null)?[]:JSON.parse(payment.payment_cards);
						const categorySlug = (payment.categories_title) ? (payment.categories_title.title_translations[this.langsConvert[this.$i18n.locale]])?payment.categories_title.title_translations[this.langsConvert[this.$i18n.locale]]:payment.categories_title.title_translations[this.langsConvert['en']] : 'ungrouped' ; // Use 'ungrouped' for payments without categories_slug
						if (!groupedPayments[categorySlug]) {
							groupedPayments[categorySlug] = [];
						}
						groupedPayments[categorySlug].push(payment);
					});

					// Convert groupedPayments object to array for Vue rendering
					this.groupedPaymentsArray = Object.keys(groupedPayments).map((key) => {
						return {
							categorySlug: key,
							payments: groupedPayments[key],
						};
					});
					this.paymentMethodIsLoading = false ;
				});
		},

		getPaymentImageUrl(name) {
			switch (name) {
				case "Visa":
				return "/images/Visa.png";
				case "MasterCard":
				return "/images/Master.png";
				case "American Express":
				return "/images/American_Express.png";
				case "Discover":
				return "/images/Discover.png";
				case "Maestro":
				return "/images/Maestro.png";
				case "JCB":
				return "/images/JCB.png";
				case "Diners Club":
				return "/images/Diners_Club.png";
				case "UnionPay":
				return "/images/Union_Pay.png";
				default:
				return 'No Image';
			}
		},

		fetchRegisterationProductId() {
			this.isLoading = true;
			let url = "/axios/web/products/get/ir_registeration/data";

			return axios
				.get(url)
				.then((res) => {
					this.$nextTick(() => {
						this.RegisterationProduct = res.data.item;
						this.RegisterationProductId = res.data.item.id;
						this.$nextTick(() => {
							this.isLoading = false;
						});
					});
				}).catch((err) => {
					this.isLoading = false;
					return false;
				});
		},

		// Addresses
		fetchAddresses() {
			this.AddressisLoading = true;
			return axios // TODO: change this to use frontend api that check for status
				.get("/axios/user/addresses/get/data", {
					params: {
						per_page: 15,
						search_key: this.searchKey,
					},
				})
				.then((response) => {
					this.addresses = response.data.items;
					this.countries = response.data.countries;
					this.address.country_id = response.data.user_country_id
					this.fetchCountryCites(this.address.country_id);
					this.fetchForm(this.address.country_id);
					this.AddressisLoading = false ;

					this.addresses.map(e => {
						if (e.default_status == 1) {
							this.selectedAddress = e;
							this.fetchShippingTypes();
							this.MoveToNext(1);
						}
					})

					this.chatBoxNum = 1;
					this.startChatBoxAnimation(1);
				})
				.catch((err) => {
					this.error.message = "Can't get addresses";
					this.error.thereAnError = true;
					this.AddressisLoading = false;
				});
		},

		fetchForm(countryId) {

			return axios
				.get("/axios/forms/get/form", {
					params: {
						country_id: countryId,
						type_id: 5
					},
				})
				.then((res) => {
					this.templateForm = res.data.form.body
				});
		},

		fetchCountryCites(countryId) {
			// this.isLoading = true;
			this.cities = {};
			return axios
				.get("/axios/user/country/cities/get/data", {
					params: {
						country_id: countryId,
					},
				})
				.then((response) => {
					this.cities = response.data.cities;
					this.address.city_id = this.cities[0] ? this.cities[0].id : null;
					this.isLoading = false;
				})
				.catch((err) => {
					this.error.message = "Can't get Countries";
					this.error.thereAnError = true;
					this.isLoading = false;
				});;
		},

		fetchShippingTypes() {

			if(this.hideAddressAndShipping){
				return true ;
			}
			if(!this.selectedAddress){
				return true ;
			}
			this.selectedShipping = null;
			this.shippingIsLoading = true;
			const cartProductIds = Object.values(this.$store.state.cart).map(item => item.id);
			return axios
				.get("/axios/order/user/shippers/get/data",{
					params: {
						address_id: this.selectedAddress.id,
						cartProductIds: cartProductIds,
					},
				})
				.then((response) => {
					this.shippingOptions = response.data.items;
					// make payments as groups
					const groupedShipping = {};
					this.shippingOptions.forEach((shipping) => {
						// [langsConvert[$i18n.locale]]
						const categorySlug = (shipping.categories_title) ? (shipping.categories_title.title_translations[this.langsConvert[this.$i18n.locale]])?shipping.categories_title.title_translations[this.langsConvert[this.$i18n.locale]]:shipping.categories_title.title_translations[this.langsConvert['en']] : 'ungrouped' ; // Use 'ungrouped' for payments without categories_slug
					if (!groupedShipping[categorySlug]) {
							groupedShipping[categorySlug] = [];
						}
						groupedShipping[categorySlug].push(shipping);
					});

					// Convert groupedPayments object to array for Vue rendering
					this.groupedShippersArray = Object.keys(groupedShipping).map((key) => {
						return {
							categorySlug: key,
							shippingOptions: groupedShipping[key],
						};
					});

				})
				.catch((err) => {
					this.error.message = "Can't get Shipping Methods";
					this.error.thereAnError = true;
				}).finally(() => {
					this.shippingIsLoading = false;
				});
		},

		fetchExtraFees() { // change here
			return axios
				.get("/axios/order/user/extra-fees/get/data", {
					params: {
						order_type: this.$store.state.shoppingType,
						cart_instance:this.$store.state.cartTypeCurrency,
					},
				})
				.then((response) => {
					this.ExtraFeesOptions = response.data.items;
				})
				.catch((err) => {
					this.error.message = "";
					this.error.thereAnError = true;
				}).finally(() => {
				});
		},

		objectToFormData(formData, data, parentKey) {
			if (
				data &&
				typeof data === "object" &&
				!(data instanceof Date) &&
				!(data instanceof File) &&
				!(data instanceof Blob)
			) {
				if (Array.isArray(data) && data.length === 0) {
					formData.append(parentKey, "[]");
				} else if (
					Object.keys(data).length === 0 &&
					data.constructor === Object
				) {
					formData.append(parentKey, "");
				} else {
					Object.keys(data).forEach((key) => {
						this.objectToFormData(
							formData,
							data[key],
							parentKey ? `${parentKey}[${key}]` : key
						);
					});
				}
			} else {
				const value = data == null ? "" : data;
				formData.append(parentKey, value);
			}

			return formData;
		},

		createAddressModal() {
			this.updateAction = false;
			this.addressModal.show();
			this.address = {
				unit_address: "",
				unit_number: "",
				state: "",
				city: "",
				phone_number: "",
			};
			this.template_form = {}
		},

		createAddress() {
			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};
			var bodyFormData = new FormData();
			var url = '';
			if(this.updateAction){
				bodyFormData.append("id",this.address.id);
				url = `/axios/user/addresses/${this.address.id}/edit`;
			}else{
				bodyFormData.append("id",null);
				url = `/axios/user/addresses/store`;
			}

			if(this.address.default_status){
				bodyFormData.append("default_status", 1);
			}else{
				bodyFormData.append("default_status", 0);
			}
			bodyFormData.append("unit_address", this.address.unit_address);
			bodyFormData.append("unit_number", this.address.unit_number);
			bodyFormData.append("city_id", this.address.city_id);
			bodyFormData.append("city", this.address.city);
			bodyFormData.append("phone_number", this.address.phone_number);
			bodyFormData.append("country_id", this.address.country_id);
			bodyFormData.append("zip_code", this.address.zip_code);
			bodyFormData.append("notes", this.address.notes);

			let pageData = {
				template_form: this.template_form,
			};
			this.objectToFormData(bodyFormData, pageData);

			let validate =
				this.address.unit_address &&
				this.address.city_id &&
				this.address.phone_number;


			let f = this.templateForm.forEach(element => {
				element.columns.forEach(ele => {
					ele.components.forEach(e => {
						if (e.required && !this.template_form[e.name]) {

							validate = false
						}
					});
				});
			});


			if (validate && !this.createAddressLoadingMode) {
				this.createAddressLoadingMode = true;
				axios
					.post(url, bodyFormData, options)
					.then((response) => {
						this.fetchAddresses();
						this.$store
						.dispatch("setFlashMessage", {
							status: true,
							SuccessMessage:  this.$t('web._ADDRESS_ADDED_SUCCESSFULY'),
						});
						this.address = [
							{
								unit_address: "",
								unit_number: "",
								state: "",
								city: "",
								phone_number: "",
							},
						];
						this.addressModal.hide();
						this.createAddressLoadingMode = false;

					})
					.catch((error) => {
						this.createAddressLoadingMode = false;
						this.$store
							.dispatch("setFlashMessage", {
								status: true,
								ErrorMessage:  this.$t("web._SERVER_ERROR"),
							});
					});
			} else {
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						InfoMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
					});
			}
		},

 		MoveToNext(step) {
			if(this.unqualifiedProducts){
				this.checkoutCollapses.collapseOne.hide();
				this.checkoutCollapses.collapseTwo.hide();
				this.checkoutCollapses.collapseThree.hide();
				this.chatBoxNum = 1;
			}else if(step == 1){
				this.selectedShipping = null;

				if(this.selectedPayment) {
					return this.MoveToNext(2);
				}


				if(!this.selectedAddress) {
					return true ;
				}
				this.chatBoxNum = 2;
				this.checkoutCollapses.collapseTwo.show();
			} else if(step == 2){
				if(this.selectedShipping) {
					return this.MoveToNext(3);
				}

				this.chatBoxNum = 3;
				this.checkoutCollapses.collapseThree.show();
			} else if(step == 3){
				if(this.selectedShipping) {
					return this.MoveToNext(4);
				}

				this.chatBoxNum = 4;
				this.checkoutCollapses.collapseThree.hide();
			} else {

				this.checkoutCollapses.collapseOne.hide();
				this.checkoutCollapses.collapseTwo.hide();
				this.checkoutCollapses.collapseThree.hide();
				this.chatBoxNum = 4;
			}
		},

		// updateCartQuantity(key, item, increase, e) {
		// 	if (this.$store.state.cartLoading) {
		// 		return false;
		// 	}
		// 	this.$store.commit("updateCartQuantity", {
		// 		key: key,
		// 		id: item.id,
		// 		increase: increase,
		// 		number: e && e.target.value
		// 	});

		// 	// this.fetchShippingTypes();

		// 	//shipping-fees
		// },

		removeItem(key, item) { // change here
			this.$store.dispatch("removeItemFromCart", {
					key: key,
					id: item.id,
					instance: this.$store.state.cartTypeCurrency,
				})
				.then(() => {
					// This code will run after the removeItemFromCart action is completed
					setTimeout(() => {
						// this.isCartItemQualifiedForRetail();
					}, 3000);
				})
				.catch((error) => {
					// Handle errors if necessary
					console.error(error);
				});
		},

		fetchWalletData() {
			this.isLoading = true;
			return axios
				.get("/axios/user/wallet/get/data", {
					params: {
						type: this.$store.state.storeCurrencyTypeId
					},
				})
				.then((response) => {
					this.isLoading = false;
					this.userWallet = response.data.amount;
					this.exchangeRate = response.data.exchange_rate;
					this.fetchPaymentMethods();
				});
		},

		submitForm() {
			console.log("submit form");
			this.submitFormLoading = true ;
			// check if user delete the registeration product
			if (this.checkIrRegisteration) {
				console.log("submit form1");
				this.checkIfRegisterationProductOnlyInCart();
				if (!this.onlyRegisterationProductInCart) {
					this.$store
						.dispatch("setFlashMessage", {
							status: true,
							ErrorMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
						});
					return false;
				}
			}
			if(parseInt(this.total_offer_bv_the_user_alreay_has) + parseInt(this.calculateMyOfferTotalBvInCart) > 101 && parseInt(this.calculateMyOfferTotalBvInCart) > 0 && this.checkUserPackage){
				console.log("submit form2");
				this.$store
						.dispatch("setFlashMessage", {
							status: true,
							ErrorMessage:  this.$t("web._You_only_100_bv_as_offer")
						});
						return false ;
			}
			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};
			var bodyFormData = new FormData();

			// bodyFormData.append("hide_in_retail", this.hide_in_retail);
			// bodyFormData.append("order_type", 1);
			bodyFormData.append("order_type", this.$store.state.shoppingType);
			bodyFormData.append("address_id", this.selectedAddress.id);
			bodyFormData.append("payment_method_id", this.selectedPayment.id);
			bodyFormData.append("cart_instance", this.$store.state.cartTypeCurrency);
			bodyFormData.append("coupon_code", this.couponCode);
			bodyFormData.append("referral_code", this.referralCode);
			bodyFormData.append("osv_coupons", JSON.stringify(this.osvCoupons));

			if((!this.hideAddressAndShipping)){
				bodyFormData.append("shipper_id", this.selectedShipping.id);
			}

			let validate = this.$store.state.shoppingType
							&& this.selectedAddress &&
							(this.selectedShipping || this.hideAddressAndShipping) &&
							this.selectedPayment;

			// check if payment method is valid
			if (this.selectedPayment.id === 19 && this.calculateGrandTotal() > 0) {
				console.log("submit form4");
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						ErrorMessage:  this.$t("web.your_order_should_be_equal_to_0_to_proceed_with_OSV"),
					});
				return false
			}

			if (validate && !this.generateOrderLoadingMode) {
				console.log("submit form5");
				this.generateOrderLoadingMode = true;
				axios
					.post("/axios/order/store", bodyFormData, options)
					.then((response) => {
						this.generateOrderLoadingMode = false;
						this.order = response.data.order;
						this.$store.commit("fetchCart", {
							instance: this.$store.state.storeCurrencyTypeId
						});
						this.$store.commit("fetchOfferCart"); // fetch My offer Cart
						this.generatePayment();
					})
					.catch((error) => {
						this.generateOrderLoadingMode = false;
						this.error.message = "can't get the order";
						this.error.thereAnError = true;
						this.isLoading = false;
					});

			} else {
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						WarningMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
					});
			}

		},

		generatePayment() {

			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};
			var bodyFormData = new FormData();

			bodyFormData.append("order_uuid", this.order.order_uuid);

			let validate =
				this.order
				;
			// !this.generatePaymentLoadingMode
			if (validate && !this.generatePaymentLoadingMode) {
				this.generatePaymentLoadingMode = true;
				axios
					.post("/axios/order/payment/generate", bodyFormData, options)
					.then((response) => {
						let url = response.data.url;
						let paymentMethodTypeId = response.data.payment_method_type_id;
						// console.log('should redirect to pay '+url);
						if (url) {
							window.location.replace(url);
						} else {
							this.generatePaymentLoadingMode = false;
							this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage: this.$t('Error in generating transaction URL'),
								});
						}
					})
					.catch((error) => {
						// this.flashError("Error", { timeout: 3000 });
						this.error.message = "can't generate payment!!";
						// this.error.thereAnError = true;
						this.isLoading = false;
						this.generatePaymentLoadingMode = false;
						this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage: this.$t("web._SERVER_ERROR"),
								});
					});
			} else {
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						WarningMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
					});
			}
		},

		addRegisterationProduct(){
			this.addRegisterationProductLoading = true ;
			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};
			var bodyFormData = new FormData();

			bodyFormData.append("instance", this.$store.state.storeCurrencyTypeId);
			axios
				.post("/axios/cart/destroy-all", bodyFormData, options)
				.then((response) => {
					this.$nextTick(() => {
						if (this.RegisterationProductId == null) {
							this.fetchRegisterationProductId();
						}
					});
					this.$nextTick(() => {
						this.addRegisterationProductToCart();
						this.$nextTick(() => {
							this.$router.go();
						});
						this.addRegisterationProductLoading = false;
					});
				})
				.catch((error) => {
					this.$store
					.dispatch("setFlashMessage", {
						status: true,
						ErrorMessage:  this.$t("web._SERVER_ERROR"),
					});
					// this.$swal({
					// 	title: this.$t("web._SERVER_ERROR"),
					// 	type: "error",
					// 	icon: "error",
					// 	position: "top-right",
					// });
					this.addRegisterationProductLoading = false;
				});
		},

		addRegisterationProductToCart(quantity = 1, instance = this.$store.state.cartTypeCurrency) {
			this.$store
				.dispatch("addProductToCart", {
					product: this.RegisterationProduct,
					quantity: quantity,
					instance: instance,
				})
				.then((response) => {
					if (!response.data.status) {
						if (response.data.needIrActivation) {
							this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage:  this.$t("web._Sorry_your_account_is_not_activated"),
								});
							this.$store
								.dispatch("showIrPopUpBanner", {
									status: true,
								});
						}
					}
				})
				.catch(() => {
					this.$store
					.dispatch("setFlashMessage", {
						status: true,
						ErrorMessage:  this.$t("web._SERVER_ERROR"),
					});
				});
		},

		calculateSelectedPaymentFees() {
			var amount = 0.0
			if (this.selectedPayment) {
				if (this.calculateSubTotalTotal() && parseInt(this.selectedPayment.sale_fees_calculation_type) === 1) {
					amount = (this.calculateSubTotalTotal() * this.selectedPayment.sale_fees_value) / 100;
				} else {
					amount = this.selectedPayment.sale_fees_value_in_exchange_rate
				}
			}

			return amount;
		},

		calculatePaymentFees(payment) {
			var amount = 0.0
			if (payment) {
				if (this.calculateSubTotalTotal() && parseInt(payment.sale_fees_calculation_type) === 1) {
					amount = (this.calculateSubTotalTotal() * payment.sale_fees_value) / 100;
				} else {
					amount = payment.sale_fees_value_in_exchange_rate
				}
			}

			return amount;
		},

		calculateShippingFees() {
			var amount = 0.0
			if (this.selectedShipping) {
				if (parseInt(this.calculateSubTotalTotal() && this.selectedShipping.shipping_fees_calculation_type) === 1) {
					amount = (this.calculateSubTotalTotal() * this.selectedShipping.shipping_fees_value_in_exchange_rate) / 100;
				} else if (parseInt(this.calculateSubTotalTotal() && this.selectedShipping.shipping_fees_calculation_type) === 2) {
					amount = this.selectedShipping.shipping_fees_value_in_exchange_rate
				} else if (parseInt(this.calculateSubTotalTotal() && this.selectedShipping.shipping_fees_calculation_type) === 3) {
					amount = this.selectedShipping.shipping_weight_fees_value_in_exchange_rate;
					this.expected_shipping_days = this.selectedShipping.expected_days;
				}
			}

			return amount;
		},

		calculateExtraFees(fees) {
			var amount = 0.0
			if (parseInt(this.calculateSubTotalTotal() && fees.extra_fees_calculation_type) === 1) {
					// amount = (this.calculateSubTotalTotal() * fees.extra_fees_value) / 100;
					amount = fees.extra_fees_value_in_exchange_rate;
				} else if (parseInt(this.calculateSubTotalTotal() && fees.extra_fees_calculation_type) === 2) {
					amount = fees.extra_fees_value_in_exchange_rate
				}
			return amount;
		},

		calculateTotalExtraFees() {
			var amount = 0.0;
			if (this.ExtraFeesOptions && this.ExtraFeesOptions.length > 0) {
				this.ExtraFeesOptions.forEach((fee) => {
					amount += this.calculateExtraFees(fee);
				});
			}
			return amount;
		},

		calculateRetailDiscountFees() {
			var amount = 0.0
			if ((!this.ShowRetailAndDistrebuteStore || this.$store.state.shoppingType == 2) && parseInt(this.$store.state.storeCurrencyTypeId) === 2) {
				var value = this.$store.state.personalRetailCommission.value;
				var type = this.$store.state.personalRetailCommission.type;
				this.calculateSubTotalTotal();
				if (type == '%') {
					amount = (this.physicalProductSubTotal * value) / 100;
				} else {
					amount = value
				}
			}
			return amount;
		},

		calculateSubTotalTotal() { // change here
			const keys = Object.keys(this.$store.state.cart);
			var amount = 0;
			var physicalAmount = 0;
			var count = 0;
			keys.forEach((key) => {
				var item = this.$store.state.cart[key];
				if (item && this.isCartItemQualifiedForCheckout(item)) {
					var price = item.options.sale_price_in_exchange_rate;
					if (item.options.have_discount) {
						price = item.options.sale_price_with_discount_in_exchange_rate;
					}
					if(item.options.type_id == 3 ){
						physicalAmount +=  price * item.qty;
					}
					amount += price * item.qty;
					count++;
				}
			});
			// calculate My Offer Cart
			const offerkeys = Object.keys(this.$store.state.offerCart);
			var offerAmount = 0.0;
			var offerPhysicalAmount = 0.0;
			var offerCount = 0;
			offerkeys.forEach((key) => {
				var offerItem = this.$store.state.offerCart[key];
				if (offerItem && this.isCartItemQualifiedForCheckout(offerItem)) {
					var offerPrice = offerItem.options.sale_price_in_exchange_rate;
					// if (offerItem.options.have_discount) {
					// 	offerPrice = offerItem.options.sale_price_with_discount_in_exchange_rate;
					// }
					if(offerItem.options.type_id == 3 ){
						offerPhysicalAmount +=  offerItem.options.sale_price_with_discount_in_exchange_rate * offerItem.qty;
					}
					offerAmount += offerPrice * offerItem.qty;
					offerCount++;
				}
			});
			this.physicalProductSubTotal = physicalAmount + offerPhysicalAmount ;
			this.shoppingTypeCount = count + offerCount;
			return amount || offerAmount ? amount + offerAmount : 0.00;
		},

		calculateTotalMyOfferDiscount() { // change here
			// calculate My Offer Cart
			const offerkeys = Object.keys(this.$store.state.offerCart);
			var offerDisscountAmount = 0;
			offerkeys.forEach((key) => {
				var offerItem = this.$store.state.offerCart[key];
				if (offerItem && this.isCartItemQualifiedForCheckout(offerItem)) {
					var offerPrice = 0;
					if (offerItem.options.have_discount) {
						offerPrice = (offerItem.options.sale_price_in_exchange_rate - offerItem.options.sale_price_with_discount_in_exchange_rate) * offerItem.qty;
					}
					offerDisscountAmount += offerPrice;
				}
			});
			return offerDisscountAmount ? offerDisscountAmount : 0.00;
		},


		calculateBVTotal() {
			const keys = Object.keys(this.$store.state.cart);
			var amount = 0;
			keys.forEach((key) => {
				var item = this.$store.state.cart[key];
				if (item && this.isCartItemQualifiedForCheckout(item)) {
					var bv = item.options.sale_bv;
					if (item.options.have_discount) {
						bv = item.options.sale_bv_with_discount;
					}
					amount += bv * item.qty;
				}
			});

			// calculate My Offer Cart
			const offerkeys = Object.keys(this.$store.state.offerCart);
			var offerAmount = 0;
			offerkeys.forEach((key) => {
				var offerItem = this.$store.state.offerCart[key];
				if (offerItem && this.isCartItemQualifiedForCheckout(offerItem)) {
					var offerBv = offerItem.options.sale_bv;
					if (offerItem.options.have_discount) {
						offerBv = offerItem.options.sale_bv_with_discount;
					}
					offerAmount += offerBv * offerItem.qty;
				}
			});

			return amount || offerAmount ? Math.round(amount + offerAmount) : 0.00;
		},

		calculateCouponDiscount() {
			var requiredDiscountAmount = this.calculateSubTotalTotal() + this.calculateShippingFees() + this.calculateTotalExtraFees() + this.calculateSelectedPaymentFees() - this.calculateRetailDiscountFees();
			var achievedDiscountAmount = 0.0;
			// requiredDiscountAmount = 90
			// coupon1 = 50
			// coupon2 = 50
			// coupon3 = 50
			this.osvCoupons.forEach((coupon, index) => {
				// coupon1 = 50
				//requiredDiscountAmount = 90 -50 = 40
				//achievedDiscountAmount = 50
				// required from this step  = requiredDiscountAmount - achievedDiscountAmount;
				var requiredDiscountForThisStep = requiredDiscountAmount - achievedDiscountAmount; // 90
				var amountToTake = 0;
				var cAmount = coupon.amount * this.exchangeRate;
				if (achievedDiscountAmount < requiredDiscountAmount) { // go in
					if (requiredDiscountForThisStep >= cAmount) {
						// take all amount
						amountToTake = cAmount; // 50
					} else if (cAmount > requiredDiscountForThisStep) {
						amountToTake = requiredDiscountForThisStep;
					}
				}
				achievedDiscountAmount += amountToTake; //50

				// update coupun data

			});

			return achievedDiscountAmount;

			// // let amount = this.calculateSubTotalTotal() * this.couponCodeDiscount;
			// let amount = 0.0;
			// if (parseInt(this.couponCodeDiscountAmountType) === 1){
			// 	amount = (this.calculateSubTotalTotal() * this.couponCodeDiscountAmount) / 100;
			// } else if (parseInt(this.couponCodeDiscountAmountType) === 2) {
			// 	amount = this.couponCodeDiscountAmount * this.exchangeRate; // 100
			// };
			// if (amount > this.calculateSubTotalTotal()){
			// 	amount = this.calculateSubTotalTotal();
			// }
			// console.log('discouht is '+amount)
			// if (this.couponCodeTypeId === 15){ // osv type
			// 	// add shipping fees and payment fees
			// 	var newAmount = amount + this.calculateShippingFees() + this.calculateSelectedPaymentFees(); //120
			// 	if (newAmount > this.couponCodeDiscountAmount * this.exchangeRate){
			// 		amount = this.couponCodeDiscountAmount * this.exchangeRate;
			// 	} else {
			// 		amount = newAmount;
			// 	}
			// }
			// return amount ? amount : 0.00;
		},

		calculateReferralDiscount() {
			let amount = this.calculateSubTotalTotal() * this.referralCodeDiscount;
			return amount ? amount : 0.00;
		},

		calculateGrandTotal() {
			var amount = this.calculateSubTotalTotal() + this.calculateSelectedPaymentFees() + this.calculateShippingFees() + this.calculateTotalExtraFees() - this.calculateRetailDiscountFees() - this.calculateCouponDiscount() - this.calculateReferralDiscount() - this.calculateTotalMyOfferDiscount();
			var finalAmount = amount ? amount : 0.00;
			return finalAmount <= 0 ? 0.0 : finalAmount;
		},

		isCartItemQualifiedForCheckout(item) {
			if(this.ShowRetailAndDistrebuteStore){
				if (this.$store.state.shoppingType === 2) {
					return true;
				} else if (this.$store.state.shoppingType === 1) {
					if (parseInt(item.options.profit_status) === 1) {
						return true;
					}
					return false;
				}
				return false;
			}
			return true ;
		},

		showCouponInput() {
			this.isCouponInputVisible = !this.isCouponInputVisible;
			$('.bs-example-modal-osv-center').modal('show');
		},

		hideCouponInput() {
			if (this.couponCode == '')
				this.isCouponInputVisible = false;
		},

		addNewOSVCoupon(event) {
			// const newValue = event.target.value;
			const newValue = this.addOsvCoupon;
			if (newValue.length >= 6) {
				var newCoupon = {
					code: newValue,
					amount: 0,
					used: 0,
					remaining: 0,
					status: 0,
				};
				this.osvCoupons.push(newCoupon);
				this.validateOSVCode(newCoupon);
				this.addOsvCoupon = '';
			}
		},

		deleteOSVCoupon(item) {
			this.removeItemFromArray(this.osvCoupons, item, 0)
		},

		removeItemFromArray(collection, item, minimumCollectionSize = 0) {
			if (collection.length <= minimumCollectionSize) {
				this.flashWarning('you should have at least one record', { timeout: 3000 });
			} else {
				const index = collection.indexOf(item);
				if (index > -1) {
					collection.splice(index, 1); // 2nd parameter means remove one item only
				}
				// this.flashSuccess('record removed successfully', { timeout: 3000 });
				this.$store
						.dispatch("setFlashMessage", {
							status: true,
							SuccessMessage:  this.$t('web._RECORD_REMOVED_SUCCESSFULLY'),
						});
			}

			this.$forceUpdate();
		},

		isValidOSVCoupon(item) {
			// lets consider the code 123 has 30% discount

			// check if code is valid
			let validate = false;
			this.isCouponLoadingMode = true;


			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};
			var bodyFormData = new FormData();
			bodyFormData.append("coupon_code", item);

			if (this.couponCode && this.couponCode.length > 0) {
				validate = true;
			}
			// !this.generatePaymentLoadingMode
			if (validate) {
				axios
					.post("/axios/coupons/validate", bodyFormData, options)
					.then((response) => {
						this.isCouponLoadingMode = false;
						//
						// 	code:'XFFDGT',
						// 		amount:10,
						// 		used:10,
						// 		remaining:10,
						// 		status:1,
						// },
						// 	if (response.data.is_valid === true){
						// 		this.couponCodeDiscountAmount = response.data.amount;
						// 		this.couponCodeDiscountAmountType = response.data.calculation_type;
						// 		this.exchangeRate = response.data.exchange_rate;
						// 		this.couponCodeTypeId = response.data.coupon_type_id;
						// 		this.$swal({
						// 			title: this.$t("web._VALID_DISCOUNT_CODE"),
						// 			type: "success",
						// 			icon: "success",
						// 			position: "top-right",
						// 		});
						// 	} else {
						// 		this.$swal({
						// 			title: this.$t("web."+response.data.msg),
						// 			type: "error",
						// 			icon: "error",
						// 			position: "top-right",
						// 		});
						// 	}
						// redirect
						// window.location.replace('/');
					})
					.catch((error) => {
						// this.flashError("Error", { timeout: 3000 });
						this.error.message = 'an error occurred';
						// this.error.thereAnError = true;
						this.isLoading = false;

						this.generatePaymentLoadingMode = false;
						this.$store
							.dispatch("setFlashMessage", {
								status: true,
								ErrorMessage:  this.$t("web._SERVER_ERROR"),
							});
						// this.$swal({
						// 	title: this.$t("web._SERVER_ERROR"),
						// 	type: "error",
						// 	icon: "error",
						// 	position: "top-right",
						// });
					});
			} else {
				this.isCouponLoadingMode = false;
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						InfoMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
					});
				// this.$swal({
				// 	title: this.$t("web._PLEASE_FILL_ALL_FIELDS"),
				// 	type: "error",
				// 	icon: "error",
				// 	position: "top-right",
				// });
			}

		},

		validateOSVCode(item) {
			// lets consider the code 123 has 30% discount

			// check if code is valid
			let validate = false;
			this.isCouponLoadingMode = true;

			if(item.code.length != 6  || item.status == 1){
				return false ;
			}

			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};
			var bodyFormData = new FormData();
			bodyFormData.append("coupon_code", item.code);

			// check if code exist
			var codeCount = 0;
			this.osvCoupons.forEach((coupon, index) => {
				if (coupon.code === item.code) {
					codeCount++;
				}
			});

			if (codeCount > 1) {
				item.code = '';
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						ErrorMessage:  this.$t("web._OSV_voucher_is_already_exist"),
					});
				return false;
			}

			if (item.code && item.code.length > 0) {
				validate = true;
			}
			// !this.generatePaymentLoadingMode
			this.osvCouponInCheckingCode = item.code;

			if (validate) {
				axios
					.post("/axios/coupons/validate", bodyFormData, options)
					.then((response) => {
						this.osvCouponInCheckingCode = null;
						this.exchangeRate = response.data.exchange_rate;
						item.amount = response.data.amount;
						item.status = response.data.is_valid === true ? 1 : 2;

						if (response.data.is_valid === true) {
							this.$store
								.dispatch("setFlashMessage", {
									status: true,
									SuccessMessage: this.$t("web._VALID_DISCOUNT_CODE"),
								});
								let grandTotal = this.calculateGrandTotal();
								if(grandTotal == 0 ){
									this.selectedPayment = this.payments.filter(e => e.id == 19)[0];
									// this.paymentIsRestricted = true ;
									this.disabledOsvPaymentMethod = false;
									this.MoveToNext(2);
								}
								else if (grandTotal <= this.userWallet) {
									if (this.$store.state.storeCurrencyTypeId && parseInt(this.$store.state.storeCurrencyTypeId) === 2) {
										this.payments.filter(e => e.id == 1)[0].disabled = false;
									} else {
										this.payments.filter(e => e.id == 2)[0].disabled = false;
									}
								}
						} else {
							this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage:  this.$t("web." + response.data.msg),
								});
						}
					})
					.catch((error) => {
						this.error.message = this.$t("web._an_error_occurred");
						this.isLoading = false;

						this.osvCouponInCheckingCode = null;
						this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage:   this.$t("web._SERVER_ERROR"),
								});
					});
			} else {
				this.osvCouponInCheckingCode = null;
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						InfoMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
					});
			}

		},

		editCouponCode() {
			// lets consider the code 123 has 30% discount

			// check if code is valid
			let validate = false;
			this.isCouponLoadingMode = true;


			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};
			var bodyFormData = new FormData();
			bodyFormData.append("coupon_code", this.couponCode);

			if (this.couponCode && this.couponCode.length > 0) {
				validate = true;
			}
			// !this.generatePaymentLoadingMode
			if (validate) {
				axios
					.post("/axios/coupons/validate", bodyFormData, options)
					.then((response) => {
						this.isCouponLoadingMode = false;
						if (response.data.is_valid === true) {
							this.couponCodeDiscountAmount = response.data.amount;
							this.couponCodeDiscountAmountType = response.data.calculation_type;
							this.exchangeRate = response.data.exchange_rate;
							this.couponCodeTypeId = response.data.coupon_type_id;
								this.$store
								.dispatch("setFlashMessage", {
									status: true,
									SuccessMessage:  this.$t("web._VALID_DISCOUNT_CODE"),
								});
						} else {
							this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage:  this.$t("web." + response.data.msg),
								});
						}
						// redirect
						// window.location.replace('/');
					})
					.catch((error) => {
						this.error.message = this.$t("web._an_error_occurred");
						this.isLoading = false;

						this.generatePaymentLoadingMode = false;
						this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage:  this.$t("web._SERVER_ERROR"),
								});

					});
			} else {
				this.isCouponLoadingMode = false;
				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						InfoMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
					});
			}

		},

		showReferralInput() {
			this.isReferralInputVisible = true;
		},

		hideReferralInput() {
			if (this.referralCode == '')
				this.isReferralInputVisible = false;
		},

		editReferralCode() {
			let validate = false;
			this.isCouponReferralLoadingMode = true;

			let options = {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			};

			// lets consider the code 123 has 30% discount referralCode
			if (this.referralCode && this.referralCode.length == 6) {
				validate = true;
			}
			// !this.generatePaymentLoadingMode
			if (validate) {
				var bodyFormData = new FormData();
				bodyFormData.append("coupon_code", this.referralCode);

				axios
					.post("/axios/coupons/referral/validate", bodyFormData, options)
					.then((response) => {
						this.isCouponReferralLoadingMode = false;
						if (response.data.is_valid === true) {
							this.referralCodeOwnerEmail = response.data.owner_email;
							this.referralCodeIsValid = true;
							this.$store
								.dispatch("setFlashMessage", {
									status: true,
									SuccessMessage: this.$t("web._referral_validated_for") + ' ' + this.referralCodeOwnerEmail,
								});
						} else {
							this.isCouponReferralLoadingMode = false;
							this.referralCodeIsValid = false;
							this.$store
								.dispatch("setFlashMessage", {
									status: true,
									ErrorMessage: this.$t("web." + response.data.msg),
								});
						}
					})
					.catch((error) => {
						this.error.message = this.$t("web._an_error_occurred");
						this.isLoading = false;
						this.isCouponReferralLoadingMode = false;
						this.$store
						.dispatch("setFlashMessage", {
							status: true,
							ErrorMessage: this.$t("web._SERVER_ERROR"),
						});
					});
			} else {
				this.isCouponReferralLoadingMode = false;

				this.$store
					.dispatch("setFlashMessage", {
						status: true,
						InfoMessage:  this.$t("web._PLEASE_FILL_ALL_FIELDS"),
					});
			}

		},

		hasTickets() {
			// Iterate through the cartData object
			for (const key in this.$store.state.cart) {
				if (this.$store.state.cart[key].options && this.$store.state.cart[key].options.attributes_values) {
				const attributes = this.$store.state.cart[key].options.attributes_values;
				// Check if any attribute has a slug with the value "gift-ticket-id"
				for (const attrKey in attributes) {
					if (attributes[attrKey].slug === "bavvo-offer" && attributes[attrKey].value != null  && attributes[attrKey].value != "" && attributes[attrKey].value == 1) {
						this.tickets.bavo = true;
					}
					if (attributes[attrKey].slug === "gift-ticket-id" && attributes[attrKey].value != null  && attributes[attrKey].value != "") {
						this.tickets.academy = true;
					}
				}
				}
			}

			// check if Offer cart has ticket
			for (const offerkey in this.$store.state.offerCart) {
				if (this.$store.state.offerCart[offerkey].options && this.$store.state.offerCart[offerkey].options.attributes_values) {
				const offerAttributes = this.$store.state.offerCart[offerkey].options.attributes_values;
				// Check if any attribute has a slug with the value "gift-ticket-id"
				for (const offerAttrKey in offerAttributes) {
					if (offerAttributes[offerAttrKey].slug === "bavvo-offer" && offerAttributes[offerAttrKey].value != null  && offerAttributes[offerAttrKey].value != "" && offerAttributes[offerAttrKey].value == 1) {
						this.tickets.bavo = true;
					}
					if (offerAttributes[offerAttrKey].slug === "gift-ticket-id" && offerAttributes[offerAttrKey].value != null  && offerAttributes[offerAttrKey].value != "") {
						this.tickets.academy = true;
					}
				}
				}
			}
			return false; // Set ticket to false if not found
		},

		scrollToDiv(q) {
			let element = document.querySelector(q);
			setTimeout(() => {
				// element.scrollIntoView({ behavior: 'smooth', offsetTop: -200 });
				// requestAnimationFrame(() => {
				// 	window.scrollBy(0, -window.innerHeight * 0.1);
				// });
				// var element = document.getElementById('targetElement');
				var headerOffset = 124;
				var elementPosition = element.getBoundingClientRect().top;
				var offsetPosition = elementPosition + window.pageYOffset - headerOffset;

				window.scrollTo({
						top: offsetPosition,
						behavior: "smooth"
				});

			}, 400);
		},

		startChatBoxAnimation(num) {

			if(  this.ShowRetailAndDistrebuteStore &&  this.$store.getters.productIsNotQualifiedForCheckout.length > 0){
				this.chatBoxMessage = `${this.$t("web._retail_unqualified")}`;
				const productNames = this.$store.getters.productIsNotQualifiedForCheckout.map(product => product);
				const unqualifiedProductsMessage = productNames.join(', ');
				this.chatBoxMessage = `${this.$t("web._retail_unqualified")}:<br><p class="tw-font-bold"> ${unqualifiedProductsMessage}<p>`;

			}else if( this.ShowRetailAndDistrebuteStore && this.$store.getters.productIsNotQualifiedForDistribute.length > 0){
				this.chatBoxMessage = `${this.$t("web._distribute_unqualified")}`;
				const productNames = this.$store.getters.productIsNotQualifiedForDistribute.map(product => product);
				const unqualifiedProductsMessage = productNames.join(', ');

				this.chatBoxMessage = `${this.$t("web._distribute_unqualified")}:<br><p class="tw-font-bold"> ${unqualifiedProductsMessage}<p>`;
			}else{
				switch(num) {
					case 1: this.chatBoxMessage = this.addresses.length == 0 ? `${this.$t("web._ADD_THE_ADDRESS_")}` : `${this.$t("web._SELECT_THE_ADDRESS")}`; break;
					case 2: this.chatBoxMessage = `${this.$t("web._SELECT_THE_PAYEMNT")}`; break;
					case 3: this.chatBoxMessage = `${this.$t("web._SELECT_THE_SHIPPER")}`; break;
					case 4: this.chatBoxMessage = `${this.$t("web._CURRENT_USER_SHOPPING")}: <b>${this.authUser?.email}</b>`; break;
				}
			}
			this.stepNum = num ;
			// reset animation
			this.visibleChatBoxMessage = '';
			this.currentCharIndex = 0;

			// this.chatBoxMessage = ;

      const intervalId = setInterval(() => {
        this.visibleChatBoxMessage = this.chatBoxMessage.slice(0, this.currentCharIndex + 1);
				this.currentCharIndex++;
        if (this.currentCharIndex === this.chatBoxMessage.length) {
          clearInterval(intervalId);
          // this.toggleCursor(); // Stop cursor blinking after animation is complete
        }
      }, 75); // Adjust the interval as needed

      // const intervalId = setInterval(() => {
      //   this.currentCharIndex++;
			// 	this.visibleChatBoxMessage = this.chatBoxMessage.slice(0, this.currentCharIndex + 1);
      //   this.currentCharIndex++;
      //   if (this.currentCharIndex === this.chatBoxMessagex.length) {
      //     clearInterval(intervalId);
      //   }

      // }, 100); // Adjust the interval as needed

   		 },

		toggleCursor() {
			setInterval(() => {
				this.showCursor = !this.showCursor;
			}, 500); // Adjust the interval to control cursor blinking speed
		},

		openTermsAndCondition(){
			if(this.contractData.use_file_as_contract == 1){
				this.contractTemplete = this.contractData.file ;
				window.open(this.contractTemplete, '_blank');
			}else{
				this.contractTemplete = this.contractData.template
				$('.bs-example-modal-terms-of-use').modal('show');
			}
		},

		agreeToTerms() {
			this.read_terms_and_condidtion = 1;
		},


	},

	watch: {
		chatBoxNum: function (val) {
			this.$nextTick(() => {
				// this.scrollToDiv('#chatbox');
				this.startChatBoxAnimation(val);
			});
		},

		isLoading: function(val) {
			// if not loading
			if(!val && (!this.$store.getters.cartIsEmpty || !this.$store.getters.offerCartIsEmpty)){
				this.$nextTick(() => {
					let el1 = document.getElementById('panelsStayOpen-collapseOne');
					let el2 = document.getElementById('panelsStayOpen-collapseTwo');
					let el3 = document.getElementById('panelsStayOpen-collapseThree');
					console.log(el1, el2, el3);
					this.checkoutCollapses['collapseOne'] = new bootstrap.Collapse(el1, { toggle: false });
					this.checkoutCollapses['collapseTwo'] = new bootstrap.Collapse(el2, { toggle: false });
					this.checkoutCollapses['collapseThree'] = new bootstrap.Collapse(el3, { toggle: false });

				console.log(this.checkoutCollapses['collapseOne'], this.checkoutCollapses['collapseTwo'], this.checkoutCollapses['collapseThree']);
				});
			}
		},
		'$store.getters.cartIsEmpty':function(val) {
			if(!val){
				this.$nextTick(() => {
					let el1 = document.getElementById('panelsStayOpen-collapseOne');
					let el2 = document.getElementById('panelsStayOpen-collapseTwo');
					let el3 = document.getElementById('panelsStayOpen-collapseThree');
					console.log(el1, el2, el3);
					this.checkoutCollapses['collapseOne'] = new bootstrap.Collapse(el1, { toggle: false });
					this.checkoutCollapses['collapseTwo'] = new bootstrap.Collapse(el2, { toggle: false });
					this.checkoutCollapses['collapseThree'] = new bootstrap.Collapse(el3, { toggle: false });

				console.log(this.checkoutCollapses['collapseOne'], this.checkoutCollapses['collapseTwo'], this.checkoutCollapses['collapseThree']);
				});
			}
		},
		'$store.getters.offerCartIsEmpty':function(val) {
			if(!val){
				this.$nextTick(() => {
					let el1 = document.getElementById('panelsStayOpen-collapseOne');
					let el2 = document.getElementById('panelsStayOpen-collapseTwo');
					let el3 = document.getElementById('panelsStayOpen-collapseThree');
					console.log(el1, el2, el3);
					this.checkoutCollapses['collapseOne'] = new bootstrap.Collapse(el1, { toggle: false });
					this.checkoutCollapses['collapseTwo'] = new bootstrap.Collapse(el2, { toggle: false });
					this.checkoutCollapses['collapseThree'] = new bootstrap.Collapse(el3, { toggle: false });

				console.log(this.checkoutCollapses['collapseOne'], this.checkoutCollapses['collapseTwo'], this.checkoutCollapses['collapseThree']);
				});
			}
		},
		'$store.state.shoppingType': function(val) {
			this.startChatBoxAnimation(this.stepNum);
		},
		'$store.state.cart': function(val) {
			if(!this.submitFormLoading){
				this.selectedShipping = null ;
				this.fetchShippingTypes();
				this.MoveToNext(1);
			}
			this.startChatBoxAnimation(this.stepNum);
		},
		'$store.state.cartCount': function(val) {
			this.fetchExtraFees();
		},
		'$store.state.offerCart': function(val) {
			if(!this.submitFormLoading){
				this.selectedShipping = null ;
				this.fetchShippingTypes();
				this.MoveToNext(1);
			}
			this.startChatBoxAnimation(this.stepNum);
		},
		'$store.state.cartCount': function(val) {
			this.fetchExtraFees();
		},
	}
};
</script>

<style scoped>
.payment-image {
	width: 60px;
	border-radius: 5px;
}


.table-container {
	overflow-x: auto;
	white-space: nowrap;
}


.summery-card {
	border-radius: 5px;
}

.step-box {
	width: 45px;
	height: 45px;
	box-sizing: border-box;
}

.step-box.active {
	background-color: var(--un-text-dark-medium);
	color: white;
}

.step-name {
	color: var(--regular-text-color);
}

tr {
	height: 30px;
}

.unset-height {
	height: unset;
}

.flex-1 {
	flex: 1;
}

input[type="radio"] {
	/* Add if not using autoprefixer */
	-webkit-appearance: none;
	/* Remove most all native input styles */
	appearance: none;
	/* For iOS < 15 */
	background-color: var(--form-background);
	/* Not removed via appearance */
	margin: 0;

	font: inherit;
	color: currentColor;
	width: 16px;
	height: 16px;
	border: 0.15em solid currentColor;
	border-radius: 50%;
	/* transform: translateY(-8px); */

	display: grid;
	place-content: center;
}

input[type="radio"]::before {
	content: "";
	width: 8px;
	height: 8px;
	border-radius: 50%;
	transform: scale(0);
	transition: 120ms transform ease-in-out;
	background-color: var(--dark-color);
	/* box-shadow: inset 1px 1px var(--dark-color); */
}

input[type="radio"]:checked+span {
	color: var(--dark-color);
}

input[type="radio"]:checked::before {
	transform: scale(1);
}

.selectedItem {
	/* border: solid 1px var(--dark-color); */
	box-shadow: 0px 0px 0px 2px var(--dark-color);
}

.summary-card {
	height: fit-content;
}

/* shopping type styles */
.top-img {
	height: 210px;
	object-fit: cover;
}

.action-btn-container>a {
	position: absolute;
	bottom: 24px;
	left: 50%;
	transform: translateX(-50%);
}

.primary-card {
	filter: drop-shadow(0px 4px 12px rgba(0, 0, 0, 0.1));
}

.seconary-card {
	filter: drop-shadow(0px 4px 4px rgba(0, 0, 0, 0.01));
}

.shipping-desc {
	transition: all .5s;
}

.left-enter-active,
.left-leave-active {
	transition: all 0.5s ease;
	transform: translateX(100%);
	opacity: 0;
}

.left-enter-from,
.left-leave-to {
	transition: all 0.5s ease;
	transform: translateX(-100%);
	opacity: 0;
}

.summary-labels {
	text-wrap: wrap;
	white-space: pre-wrap;
	max-width: 65%;
}

.accordion-item:not(:first-of-type) {
	border-top: 1px solid;
}

.accordion-button::after {
	/* content: ""; */
	rotate: 0 !important;
	transform: rotate(0deg) !important;
	width: 20px;
	height: 20px;
	color: #777;
	background-image: url("data:image/svg+xml,<svg width='24' height='24' viewBox='0 0 20 20' fill='none' xmlns='http://www.w3.org/2000/svg'><path d='M14.0497 3.73937L15.4555 2.33271C15.7486 2.03964 16.1461 1.875 16.5605 1.875C16.975 1.875 17.3725 2.03964 17.6655 2.33271C17.9586 2.62577 18.1233 3.02325 18.1233 3.43771C18.1233 3.85216 17.9586 4.24964 17.6655 4.54271L8.81638 13.3919C8.37582 13.8322 7.83252 14.1558 7.23555 14.3335L4.99805 15.0002L5.66471 12.7627C5.84245 12.1657 6.16608 11.6224 6.60638 11.1819L14.0497 3.73937ZM14.0497 3.73937L16.248 5.93771M14.998 11.6669V15.6252C14.998 16.1225 14.8005 16.5994 14.4489 16.951C14.0972 17.3027 13.6203 17.5002 13.123 17.5002H4.37305C3.87577 17.5002 3.39885 17.3027 3.04722 16.951C2.69559 16.5994 2.49805 16.1225 2.49805 15.6252V6.87521C2.49805 6.37793 2.69559 5.90101 3.04722 5.54938C3.39885 5.19775 3.87577 5.00021 4.37305 5.00021H8.33138' stroke='%23777777' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/></svg>") !important;
}

.cursor {
  display: inline-block;
  animation: blink-animation 1s infinite;
}

@keyframes blink-animation {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

.ref-input-parent > input {
	text-align: center;
	transition: all .3s;
}


.ref-input-parent > svg {
	opacity: 0;
	transition: all .3s;
}

.ref-input-parent:has(input:focus) {
	background-color: white;
}

.ref-input-parent:has(input:focus) > svg {
	transform: none;
	opacity: 1;
}

.ref-input-parent:has(input:focus) > input {
	text-align: start;
	padding-inline-start: 32px;
}

</style>
