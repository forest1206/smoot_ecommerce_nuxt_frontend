<template>
  <b-container class="container-sell" rounded fluid="xl">
    <div class="sell-header">
      <i class="fa fa-times cursor-pointer" aria-hidden="true"></i>
      <h1>What are you listing today?</h1>
    </div>
    <b-row>
      <!-- eslint-disable -->
      <b-col
        :lg="previewOfImages.length ? `5` : `12`"
        :md="previewOfImages.length ? `5` : `12`"
        sm="12"
        cols="12"
        class="col-left"
      >
      <!-- eslint-enable -->
        <b-row>
          <b-col cols="12">
            <div
              v-if="previewOfImages.length < 10"
              class="photos-upload-area cursor-pointer rounded"
            >
              <i class="fa fa-file-image-o fa-3x" aria-hidden="true"></i>
              <b-button size="sm">
                <i class="fa fa-plus" aria-hidden="true"></i>
                Add photos
              </b-button>
              <input
                type="file"
                class="cursor-pointer default-file-upload-field"
                accept="image/jpeg, image/png"
                @change="onProductImageSelected"
              />
              <div class="add-photo-text text-align-center">
                <div>or drag & drop photos here</div>
                <div>(Up to 10 photos)</div>
              </div>
            </div>
          </b-col>
          <b-col cols="12">
            <div class="list-uploaded-images">
              <div
                v-for="(previewOfImage, indexPreviewOfImage) in previewOfImages"
                :key="indexPreviewOfImage"
                class="image-block rounded"
              >
                <!-- eslint-disable -->
                <img
                  :src="previewOfImage.blob"
                  draggable
                  @dragstart="onDragStart($event, indexPreviewOfImage, previewOfImage.blob)"
                  @drop="onDrop($event, indexPreviewOfImage, previewOfImage.blob)"
                  @dragover.prevent
                  @dragenter.prevent
                />
                <!-- eslint-enable -->
                <i
                  class="fa fa-times-circle-o remove-image cursor-pointer"
                  aria-hidden="true"
                  @click="removeImage(indexPreviewOfImage)"
                ></i>
                <span
                  v-if="indexPreviewOfImage === 0"
                  class="product-cover text-align-center text-uppercase"
                >
                  Cover
                </span>
              </div>
            </div>
          </b-col>
          <b-col
            v-if="previewOfImages.length >= 1"
            cols="12"
            class="col-image-arrange-tip"
          >
            <div class="tip-container text-align-center">
              <i class="fa fa-lightbulb-o" aria-hidden="true"></i>
              <span>Tip: Re-arrange photos to change cover</span>
            </div>
          </b-col>
        </b-row>
      </b-col>
      <b-col
        v-if="previewOfImages.length"
        lg="7"
        md="7"
        sm="12"
        cols="12"
        class="col-right"
      >
        <b-row>
          <b-col>
            <b-form class="form-product-listing">
              <b-form-group>
                <label for="price">
                  <h2>Price</h2>
                </label>
                <b-form-radio-group
                  v-model="productListingsForm.priceOption"
                  :options="productListingsFormLookUpValues.priceTypes"
                ></b-form-radio-group>
                <b-input-group
                  v-if="productListingsForm.priceOption === `for-sale`"
                  class="input-group-for-price"
                  prepend="Rs."
                  append=".00"
                >
                  <b-form-input
                    id="price"
                    class="text-align-right"
                  ></b-form-input>
                </b-input-group>
              </b-form-group>
              <b-form-group>
                <label for="category">
                  <h3>Category</h3>
                </label>
                <!-- eslint-disable -->
                <div
                  id="category"
                  class="fake-input-category-dd-field cursor-pointer"
                  @click="openOrCloseFakeCategoryDataList('toggle-data-list')"
                >
                  <div class="default-inline-label">
                    <span v-if="!mainCategoryToSubCategryLables.length">
                      Select a category
                    </span>
                    <span class="selected-category-label" v-else>
                      <div class="category" v-if="mainCategoryToSubCategryLables.length === 1">
                        {{ mainCategoryToSubCategryLables[0] }}
                      </div>
                      <div class="category-with-sub" v-else-if="mainCategoryToSubCategryLables.length === 2">
                        <div>
                          {{ mainCategoryToSubCategryLables[1] }}
                        </div>
                        <div>
                          {{ `in ${mainCategoryToSubCategryLables[0]}` }}
                        </div>
                      </div>
                      <div class="category-with-sub" v-else>
                        <div>
                          {{ mainCategoryToSubCategryLables[2] }}
                        </div>
                        <div>
                          {{ `in ${mainCategoryToSubCategryLables[0]} > ${mainCategoryToSubCategryLables[1]}` }}
                        </div>
                      </div>
                    </span>
                    <span>
                      <i
                        class="fa fa-angle-down fa-lg"
                        :class="showCategoryDatalist ? `fa-flip-vertical` : null"
                        aria-hidden="true"
                      ></i>
                    </span>
                  </div>
                  <div v-if="showCategoryDatalist || showCategoryListForDataList" class="data-list" @click="openOrCloseFakeCategoryDataList('select-from-data-list');closeCategoryDataList()">
                    <div class="search-bar">
                      <b-form-input
                        placeholder="Search for a category..."
                        @click="keepDataListOpenForSearch()"
                      ></b-form-input>
                    </div>
                    <div
                      :id="`categories-parent-${mainCategoryIndex}`"
                      v-for="(mainCategory, mainCategoryIndex) in productListingsFormLookUpValues.categories"
                      v-b-toggle="`collapse-${mainCategoryIndex}`"
                      class="categories-list"
                      :key="mainCategoryIndex"
                    >
                      <div @click="checkIfMainCategoryHasCollapsed(mainCategoryIndex)">
                        <div class="main-category" @click="pushToSelectedCategoriesArray(mainCategory.text, mainCategory.value, `main-category`, mainCategory.hasOwnProperty(`sub_categories`))">
                          <span :id="`main-category-${mainCategoryIndex}`">
                            {{ mainCategory.text }}
                          </span>
                          <span
                            v-if="mainCategory.hasOwnProperty(`sub_categories`)"
                          >
                            <i
                            :id="`icon-${mainCategoryIndex}`"
                            class="fa fa-angle-right fa-lg"
                            aria-hidden="true"
                          ></i>
                          </span>
                        </div>
                        <!-- collapse for sub category 1 list -->
                        <b-collapse
                          :id="`collapse-${mainCategoryIndex}`"
                          v-if="mainCategory.hasOwnProperty(`sub_categories`)"
                        >
                          <div
                            :id="`categories-sub1-${mainCategoryIndex}${subCategory1Index}`"
                            v-for="(subCategory1, subCategory1Index) in mainCategory.sub_categories"
                            :key="subCategory1Index"
                            v-b-toggle="`collapse-${mainCategoryIndex} collapse-${mainCategoryIndex}${subCategory1Index}`"
                          >
                            <div @click="checkIfSubCategory1HasCollapsed(`${mainCategoryIndex}${subCategory1Index}`)">
                              <div class="sub-category" @click="pushToSelectedCategoriesArray(subCategory1.text, subCategory1.value, `sub-category-1`, subCategory1.hasOwnProperty(`sub_categories`))">
                                <span :id="`sub-1-category-${mainCategoryIndex}${subCategory1Index}`">
                                  {{ subCategory1.text }}
                                </span>
                                <span v-if="subCategory1.hasOwnProperty(`sub_categories`)">
                                  <i
                                    :id="`icon-${mainCategoryIndex}${subCategory1Index}`"
                                    class="fa fa-angle-right fa-lg"
                                    aria-hidden="true"
                                  ></i>
                                </span>
                              </div>
                              <!-- collapse for sub category 2 list -->
                              <b-collapse
                                :id="`collapse-${mainCategoryIndex}${subCategory1Index}`"
                                v-if="subCategory1.hasOwnProperty(`sub_categories`)"
                              >
                                <div
                                  v-for="(subCategory2, subCategory2Index) in subCategory1.sub_categories"
                                  :key="subCategory2Index"
                                >
                                  <div class="sub-category padding-for-sub-cat-2" @click="pushToSelectedCategoriesArray(subCategory2.text, subCategory2.value, `sub-category-2`, subCategory2.hasOwnProperty(`sub_categories`))">
                                    <span :id="`sub-2-category-${mainCategoryIndex}${subCategory1Index}${subCategory2Index}`">
                                      {{ subCategory2.text }}
                                    </span>
                                    <span v-if="subCategory2.hasOwnProperty(`sub_categories`)">
                                      <i
                                        class="fa fa-angle-right fa-lg"
                                        aria-hidden="true"
                                      ></i>
                                    </span>
                                  </div>
                                </div>
                              </b-collapse>
                              <!-- end collapse for sub category 2 list -->
                            </div>
                          </div>
                        </b-collapse>
                        <!-- end collapse for sub category 1 list -->
                      </div>
                    </div>
                  </div>
                </div>
                <!-- eslint-enable -->
              </b-form-group>
              <div v-if="productListingsForm.mainCategoryToSubCategry.length">
                <b-form-group>
                  <label for="title">
                    <h2>Listing Title</h2>
                  </label>
                  <b-form-input id="title" type="text"></b-form-input>
                </b-form-group>
                <!-- dynamic fields based on the selected category -->
                <!-- eslint-disable -->
                <div v-if="productRelatedFields.hasOwnProperty(productListingsForm.mainCategoryToSubCategry[productListingsForm.mainCategoryToSubCategry.length - 1]) && productRelatedFields[productListingsForm.mainCategoryToSubCategry[productListingsForm.mainCategoryToSubCategry.length - 1]].length">
                    <h2>Product Detail</h2>
                    <b-form-group
                      v-for="(productRelatedField, productRelatedFieldIndex) in productRelatedFields[productListingsForm.mainCategoryToSubCategry[productListingsForm.mainCategoryToSubCategry.length - 1]]"
                      :key="productRelatedFieldIndex"
                    >
                      <!-- text field -->
                      <div v-if="productRelatedField.input_type === `text`">
                        <label :for="productRelatedField.value">
                          <h3>{{ productRelatedField.label }}</h3>
                        </label>
                        <b-form-input
                          :id="productRelatedField.value"
                          type="text"
                          :placeholder="productRelatedField.placeholder"
                          ></b-form-input>
                      </div>
                      <!-- end text field -->
                      <!-- drop down field -->
                      <div v-if="productRelatedField.input_type === `ddl`">
                        <label :for="productRelatedField.value">
                          <h3>{{ productRelatedField.label }}</h3>
                        </label>
                        <b-form-select
                          :id="productRelatedField.value"
                          :options="productRelatedField.values"
                          ></b-form-select>
                      </div>
                      <!-- end drop down field -->
                      <!-- checkbox field -->
                      <div v-if="productRelatedField.input_type === `checkbox`">
                        <label :for="productRelatedField.value">
                          <h3>{{ productRelatedField.label }}</h3>
                        </label>
                        <b-form-checkbox-group
                          :id="productRelatedField.value"
                          :options="productRelatedField.values"
                        ></b-form-checkbox-group>
                      </div>
                      <!-- end checkbox field -->
                      <!-- radiobox field -->
                      <div v-if="productRelatedField.input_type === `radiobox`">
                        <label :for="productRelatedField.value">
                          <h3>{{ productRelatedField.label }}</h3>
                        </label>
                        <b-form-radio-group
                          :id="productRelatedField.value"
                          :options="productRelatedField.values"
                      ></b-form-radio-group>
                      </div>
                      <!-- end radiobox field -->
                    </b-form-group>
                </div>
                <!-- eslint-enable -->
                <!-- end dynamic fields based on the selected category -->
                <b-form-group>
                  <label for="condition">
                    <h2>Item Condition</h2>
                  </label>
                  <b-form-radio-group
                    id="condition"
                    v-model="productListingsForm.condition"
                    :options="productListingsFormLookUpValues.itemConditions"
                  ></b-form-radio-group>
                </b-form-group>
                <b-form-group>
                  <label for="description">
                    <h2>Description</h2>
                  </label>
                  <b-form-textarea
                    id="description"
                    class="description"
                    placeholder="Describe what you are selling and include any details a buyer might be interested in. People love items with stories!"
                    rows="6"
                  ></b-form-textarea>
                </b-form-group>
                <b-form-group>
                  <label>
                    <h2>Deal Method</h2>
                  </label>
                  <b-form-checkbox-group
                    id="deal-method-for-meetup"
                    v-model="productListingsForm.dealMethodForMeetup"
                    :options="
                      productListingsFormLookUpValues.dealMethodsForMeetup
                    "
                    stacked
                  ></b-form-checkbox-group>
                  <b-form-input
                    v-if="productListingsForm.dealMethodForMeetup.length"
                    id="meetup-location"
                    v-model="productListingsForm.meetupLocation"
                    class="meetup-location"
                    type="text"
                    placeholder="Type a location you want to have the meet-up"
                  ></b-form-input>
                  <b-form-checkbox-group
                    id="deal-method-for-delivery"
                    v-model="productListingsForm.dealMethod"
                    :options="
                      productListingsFormLookUpValues.dealMethodsForDelivery
                    "
                    stacked
                  ></b-form-checkbox-group>
                </b-form-group>
                <b-form-group class="mb-0">
                  <b-button type="submit" block>List now</b-button>
                </b-form-group>
              </div>
            </b-form>
          </b-col>
        </b-row>
      </b-col>
    </b-row>
  </b-container>
</template>
<script>
export default {
  name: "Sell",
  data() {
    return {
      previewOfImages: [],
      mainCategoryToSubCategryLables: [],
      productListingsForm: {
        mainCategoryToSubCategry: [],
        priceOption: "for-sale",
        price: "",
        condition: "",
        description: "",
        dealMethodForMeetup: [],
        meetupLocation: "",
        dealMethodForDelivery: [],
      },
      productListingsFormLookUpValues: {
        priceTypes: [
          {
            text: `For Sale`,
            value: `for-sale`,
          },
          {
            text: `For Rent`,
            value: `for-rent`,
          },
          {
            text: `For Free`,
            value: `for-free`,
          },
        ],
        categories: [
          {
            text: `Electronics`,
            value: `electronics`,
            sub_categories: [
              {
                text: `Audio`,
                value: `audio`,
              },
              {
                text: `Computers`,
                value: `computers`,
                sub_categories: [
                  {
                    text: `Laptops`,
                    value: `laptops`,
                  },
                  {
                    text: `Desktops`,
                    value: `desktops`,
                  },
                  {
                    text: `Others`,
                    value: `Others`,
                  },
                ],
              },
              {
                text: `Computer Parts & Accessories`,
                value: `computer-parts-and-accessories`,
              },
              {
                text: `Electronics & Gadgets - Others`,
                value: `electronics-and-gadgets-others`,
              },
            ],
          },
          {
            text: `Mobile Phones & Tablets`,
            value: `mobile-phones-and-tablets`,
            sub_categories: [
              {
                text: `iPhone`,
                value: `iphone`,
                sub_categories: [
                  {
                    text: `iPhone 11 Series`,
                    value: `iphone-11-series`,
                  },
                  {
                    text: `iPhone X Series`,
                    value: `iphone-x-series`,
                  },
                  {
                    text: `iPhone 8 Series`,
                    value: `iphone-8-series`,
                  },
                  {
                    text: `iPhone 7 Series`,
                    value: `iphone-7-series`,
                  },
                  {
                    text: `iPhone 6 Series`,
                    value: `iphone-6-series`,
                  },
                  {
                    text: `Others`,
                    value: `others`,
                  },
                ],
              },
              {
                text: `Android Phones`,
                value: `android-phones`,
                sub_categories: [
                  {
                    text: `Samsung`,
                    value: `Samsung`,
                  },
                  {
                    text: `Sony`,
                    value: `sony`,
                  },
                  {
                    text: `Xiaomi`,
                    value: `xiaomi`,
                  },
                  {
                    text: `LG`,
                    value: `lg`,
                  },
                  {
                    text: `OPPO`,
                    value: `oppo`,
                  },
                  {
                    text: `Others`,
                    value: `others`,
                  },
                ],
              },
              {
                text: `Tablets`,
                value: `tablets`,
              },
              {
                text: `Mobile & Tablet Accessories`,
                value: `mobile-and-tablet-accessories`,
              },
              {
                text: `Mobile Phones & Tablets - Others`,
                value: `mobile-phones-and-tablets-and-Others`,
              },
            ],
          },
          {
            text: `Women's Fashion`,
            value: `womens-fashion`,
            sub_categories: [
              {
                text: `Women's Watches`,
                value: `womens-watches`,
              },
              {
                text: `Women's Bags & Wallets`,
                value: `womens-bags-and-wallets`,
              },
              {
                text: `Women's Shoes`,
                value: `womens-and-shoes`,
              },
              {
                text: `Women's Jewellery`,
                value: `womens-jewellery`,
              },
              {
                text: `Women's Accessories`,
                value: `womens-and-accessories`,
              },
            ],
          },
          {
            text: `Health & Beauty`,
            value: `health-and-beauty`,
          },
          {
            text: `Food & Drinks`,
            value: `food-and-drinks`,
          },
          {
            text: `Furniture`,
            value: `furniture`,
          },
          {
            text: `Home Appliances`,
            value: `home-appliances`,
          },
          {
            text: `Babies & Kids`,
            value: `babies-and-kids`,
          },
          {
            text: `Entertainment`,
            value: `entertainment`,
          },
          {
            text: `Photography`,
            value: `photography`,
          },
        ],
        itemConditions: [
          {
            text: `New`,
            value: `new`,
          },
          {
            text: `Used`,
            value: `used`,
          },
        ],
        dealMethodsForMeetup: [
          {
            text: `Meet-up`,
            value: `meet-up`,
          },
        ],
        dealMethodsForDelivery: [
          {
            text: `Mailing & Delivery`,
            value: `mailing-and-delivery`,
          },
        ],
      },
      productRelatedFields: {
        audio: [
          {
            input_type: "ddl",
            label: "Type",
            value: "type",
            placeholder: "",
            isRequired: true,
            values: [
              {
                text: `Earphones`,
                value: `earphones`,
              },
              {
                text: `Speaker`,
                value: `speaker`,
              },
              {
                text: `Amplifier`,
                value: `amplifier`,
              },
              {
                text: `Microphone`,
                value: `microphone`,
              },
              {
                text: `Others`,
                value: `others`,
              },
            ],
          },
          {
            input_type: "text",
            label: "Brand (Optional)",
            value: "brand",
            placeholder: "",
            isRequired: false,
          },
          {
            input_type: "checkbox",
            label: "Special Features (Optional)",
            value: "special-features",
            placeholder: "",
            isRequired: false,
            values: [
              {
                text: `Noise Cancelling`,
                value: `noise-cancelling`,
              },
              {
                text: `Wireless`,
                value: `wireless`,
              },
              {
                text: `For Fashion`,
                value: `for-fashion`,
              },
              {
                text: `For Djs`,
                value: `for-djs`,
              },
              {
                text: `Others`,
                value: `others`,
              },
            ],
          },
          // fields: {
          //   type: {
          //     input_type: "ddl",
          //     label: "Type",
          //     value: "type",
          //     placeholder: "",
          //     isRequired: true,
          //     values: [
          //       {
          //         text: `Earphones`,
          //         value: `earphones`,
          //       },
          //       {
          //         text: `Speaker`,
          //         value: `speaker`,
          //       },
          //       {
          //         text: `Amplifier`,
          //         value: `amplifier`,
          //       },
          //       {
          //         text: `Microphone`,
          //         value: `microphone`,
          //       },
          //       {
          //         text: `Others`,
          //         value: `others`,
          //       },
          //     ],
          //   },
          //   brand: {
          //     input_type: "text",
          //     label: "Brand",
          //     value: "brand",
          //     placeholder: "",
          //     isRequired: false,
          //   },
          //   special_features: {
          //     input_type: "checkbox",
          //     label: "Special Features (Optional)",
          //     value: "special-features",
          //     placeholder: "",
          //     isRequired: false,
          //     values: [
          //       {
          //         text: `Noise Cancelling`,
          //         value: `noise-cancelling`,
          //       },
          //       {
          //         text: `Wireless`,
          //         value: `wireless`,
          //       },
          //       {
          //         text: `For Fashion`,
          //         value: `for-fashion`,
          //       },
          //       {
          //         text: `For Djs`,
          //         value: `for-djs`,
          //       },
          //       {
          //         text: `Others`,
          //         value: `others`,
          //       },
          //     ],
          //   },
          // },
        ],
        "iphone-11-series": [
          {
            input_type: "radiobox",
            label: "Model",
            value: "model",
            placeholder: "",
            isRequired: true,
            values: [
              {
                text: `iPhone 11`,
                value: `iphone-11`,
              },
              {
                text: `iPhone 11 Pro`,
                value: `iphone-11-pro`,
              },
              {
                text: `iPhone 11 Pro Max`,
                value: `iphone-11-pro-max`,
              },
            ],
          },
          {
            input_type: "radiobox",
            label: "Storage",
            value: "storage",
            placeholder: "",
            isRequired: true,
            values: [
              {
                text: `64 GB`,
                value: `64-gb`,
              },
              {
                text: `128 GB`,
                value: `128-gb`,
              },
              {
                text: `256 GB`,
                value: `256-gb`,
              },
              {
                text: `512 GB`,
                value: `512-gb`,
              },
            ],
          },
          {
            input_type: "radiobox",
            label: "Color",
            value: "color",
            placeholder: "",
            isRequired: true,
            values: [
              {
                text: `Black`,
                value: `black`,
              },
              {
                text: `White`,
                value: `white`,
              },
              {
                text: `Red`,
                value: `red`,
              },
              {
                text: `Green`,
                value: `green`,
              },
            ],
          },
        ],
        "iphone-7-series": [
          {
            input_type: "radiobox",
            label: "Model",
            value: "model",
            placeholder: "",
            isRequired: true,
            values: [
              {
                text: `iPhone 7 Plus`,
                value: `iphone-7-plus`,
              },
              {
                text: `iPhone 7`,
                value: `iphone-7`,
              },
            ],
          },
          {
            input_type: "radiobox",
            label: "Storage",
            value: "storage",
            placeholder: "",
            isRequired: true,
            values: [
              {
                text: `64 GB`,
                value: `64-gb`,
              },
              {
                text: `128 GB`,
                value: `128-gb`,
              },
              {
                text: `256 GB`,
                value: `256-gb`,
              },
            ],
          },
          {
            input_type: "radiobox",
            label: "Color",
            value: "color",
            placeholder: "",
            isRequired: true,
            values: [
              {
                text: `Jet Black`,
                value: `jet-black`,
              },
              {
                text: `Black`,
                value: `black`,
              },
              {
                text: `Silver`,
                value: `silver`,
              },
              {
                text: `Gold`,
                value: `gold`,
              },
              {
                text: `Rose Gold`,
                value: `rose-gold`,
              },
              {
                text: `Red`,
                value: `red`,
              },
            ],
          },
        ],
      },
      showCategoryDatalist: false,
      showCategoryListForDataList: false,
    };
  },
  watch: {},
  methods: {
    onProductImageSelected(event) {
      if (typeof event.target.files[0] !== "undefined") {
        this.previewOfImages.push({
          blob: window.URL.createObjectURL(event.target.files[0]),
        });
      }
    },
    removeImage(removeIndex) {
      const vm = this;
      this.$delete(vm.previewOfImages, removeIndex);
    },
    onDragStart(event, dragdingImageIndex, dragdingImage) {
      event.dataTransfer.dropEffect = "move";
      event.dataTransfer.effectAllowed = "move";
      event.dataTransfer.setData("imageIndex", dragdingImageIndex);
      event.dataTransfer.setData("image", dragdingImage);
    },
    onDrop(event, indexToPlaceTheDraggingItem, existngImageInTheDroppingIndex) {
      const vm = this;
      const imageIndex = event.dataTransfer.getData("imageIndex");
      const image = event.dataTransfer.getData("image");
      vm.previewOfImages[indexToPlaceTheDraggingItem].blob = image;
      vm.previewOfImages[imageIndex].blob = existngImageInTheDroppingIndex;
    },
    checkIfMainCategoryHasCollapsed(categoryListElement) {
      const elementParent = document.getElementById(
        `categories-parent-${categoryListElement}`
      );
      const elementForIcon = document.getElementById(
        `icon-${categoryListElement}`
      );
      if (elementForIcon !== null) {
        if (elementParent.classList.contains("not-collapsed")) {
          elementForIcon.classList.remove("fa-rotate-90");
        } else {
          elementForIcon.classList.add("fa-rotate-90");
        }
      }
    },
    checkIfSubCategory1HasCollapsed(categoryListElement) {
      const elementSub = document.getElementById(
        `collapse-${categoryListElement}`
      );
      const elementForIcon = document.getElementById(
        `icon-${categoryListElement}`
      );
      if (elementForIcon !== null) {
        if (elementSub.classList.contains("show")) {
          elementForIcon.classList.remove("fa-rotate-90");
        } else {
          elementForIcon.classList.add("fa-rotate-90");
        }
      }
    },
    openOrCloseFakeCategoryDataList(actionType) {
      if (actionType === "toggle-data-list") {
        this.showCategoryDatalist = !this.showCategoryDatalist;
        this.showCategoryListForDataList = false;
      } else if (actionType === "select-from-data-list") {
        // make showCategoryDatalist = false as default
        this.showCategoryDatalist = true;
        this.showCategoryListForDataList = true;
      }
    },
    closeCategoryDataList() {
      if (this.productListingsForm.mainCategoryToSubCategry.length) {
        this.showCategoryDatalist = false;
        this.showCategoryListForDataList = false;
      }
      event.stopPropagation();
    },
    keepDataListOpenForSearch() {
      this.showCategoryDatalist = true;
      event.stopPropagation();
    },
    /* eslint-disable */
    pushToSelectedCategoriesArray(categoryText, categoryValue, categoryType, hasSubCategory){
      const vm = this;
      if(vm.productListingsForm.mainCategoryToSubCategry.length){
        vm.productListingsForm.mainCategoryToSubCategry = [];
      }
      if(vm.mainCategoryToSubCategryLables.length){
        vm.mainCategoryToSubCategryLables = [];
      }
      if(categoryType === "main-category") {
        if(!hasSubCategory) {
          vm.productListingsForm.mainCategoryToSubCategry.push(
            categoryValue
          );
          vm.mainCategoryToSubCategryLables.push(
            categoryText
          );
        }
      } else if(categoryType === "sub-category-1") {
        if(!hasSubCategory) {
            /* get the main category */
            vm.productListingsFormLookUpValues.categories.filter(function(mainCategory) {
              if(mainCategory.hasOwnProperty("sub_categories")){
                mainCategory.sub_categories.filter(function(sub1Category) {
                  if(sub1Category.value === categoryValue) {
                      vm.productListingsForm.mainCategoryToSubCategry.push(
                      mainCategory.value,
                      categoryValue,
                    );
                    vm.mainCategoryToSubCategryLables.push(
                      mainCategory.text,
                      categoryText
                    );
                  }
                  return;
                });
              }
            });
          /* end get the main category */
        }
      }
      else {
        /* get the main category, sub 2 category */
        vm.productListingsFormLookUpValues.categories.filter(function(mainCategory) {
          if(mainCategory.hasOwnProperty("sub_categories")){
            mainCategory.sub_categories.filter(function(sub1Category) {
              if(sub1Category.hasOwnProperty("sub_categories")) {
                sub1Category.sub_categories.filter(function (sub2Category) {
                  if(sub2Category.value === categoryValue) {
                    vm.productListingsForm.mainCategoryToSubCategry.push(
                      mainCategory.value,
                      sub1Category.value,
                      categoryValue,
                    );
                    vm.mainCategoryToSubCategryLables.push(
                      mainCategory.text,
                      sub1Category.text,
                      categoryText
                    );
                  }
                  return;
                });
              }
            });
          }
        });
        /* get the main category, sub 2 category */
      }
    },
    /* eslint-enable */
  },
};
</script>
