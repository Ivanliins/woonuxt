<template>
  <main v-if="product" class="container relative py-6 xl:max-w-7xl">
    <!-- Breadcrumb -->
    <Breadcrumb
      class="mb-6"
      :format="[
        { name: 'Products', slug: '/products' },
        {
          name: primaryCategory.name,
          slug: `/product-category/${primaryCategory.slug}`,
        },
        { name: product.name },
      ]" />

    <div class="flex flex-col gap-10 md:flex-row md:justify-between lg:gap-24">
      <ProductImageGallery class="relative flex-1" :first-image="product.image.sourceUrl" :main-image="type.image.sourceUrl" :gallery="product.galleryImages" :node="type" />

      <div class="md:max-w-md md:py-2">
        <div class="flex justify-between mb-4">
          <div class="flex-1">
            <h1 class="mb-2 text-2xl font-sesmibold">{{ type.name }}</h1>
            <StarRating :rating="product.averageRating" :count="product.reviewCount" />
          </div>
          <ProductPrice class="text-xl" :sale-price="type.salePrice" :regular-price="type.regularPrice" />
        </div>

        <div class="grid gap-2 my-8 text-sm">
          <div class="flex items-center gap-2">
            <span class="text-gray-400">{{ $t('messages.shop.availability') }}: </span>
            <span v-if="type.stockStatus == 'IN_STOCK'" class="text-green-600">{{ $t('messages.shop.inStock') }}</span>
            <span v-else class="text-red-600">{{ $t('messages.shop.outOfStock') }}</span>
          </div>
          <div class="flex items-center gap-2">
            <span class="text-gray-400">{{ $t('messages.shop.sku') }}: </span>
            <span>{{ product.sku || 'N/A' }}</span>
          </div>
        </div>

        <div class="mb-8 font-light prose" v-html="product.description || product.shortDescription"></div>

        <hr />

        <form @submit.prevent="addToCart(selectProductInput)">
          <AttributeSelections
            v-if="product.type == 'VARIABLE' && product.attributes"
            class="mt-4 mb-8"
            :attrs="product.attributes.nodes"
            :variations="product.variations.nodes"
            @attrs-changed="updateSelectedVariations" />
          <div class="flex items-center gap-4 mt-12">
            <input
              v-model="quantity"
              type="number"
              min="1"
              aria-label="Quantity"
              class="bg-white border rounded-lg flex text-left p-2.5 w-20 gap-4 items-center justify-center focus:outline-none" />
            <AddToCartButton class="flex-1 w-full md:max-w-xs" :disabled="!activeVariation && !!product.variations" :class="{ loading: isUpdatingCart }" />
          </div>
        </form>

        <div class="grid gap-2 my-8 text-sm">
          <div class="flex items-center gap-2">
            <span class="text-gray-400">{{ $t('messages.shop.category', 2) }}:</span>
            <div class="product-categories">
              <NuxtLink v-for="category in product.productCategories.nodes" :key="category.slug" :to="`/product-category/${category.slug}`" class="hover:text-primary"
                >{{ category.name }}<span class="comma">, </span></NuxtLink
              >
            </div>
          </div>
        </div>
        <hr />
        <div class="flex flex-wrap gap-4">
          <WishlistButton :product="product" />
          <ShareButton :product="product" />
        </div>
      </div>
    </div>
    <div class="my-32">
      <ProductTabs :product="product" />
    </div>
    <div class="my-32">
      <div class="mb-4 text-xl font-semibold">{{ $t('messages.shop.youMayLike') }}</div>
      <ProductRow :products="product.related.nodes" class="grid-cols-2 md:grid-cols-4 lg:grid-cols-5" />
    </div>
  </main>
</template>

<script setup>
const route = useRoute();
const { arraysEqual, formatArray } = useHelpers();
const { addToCart, isUpdatingCart } = useCart();

const { product } = await GqlGetProduct({ slug: route.params.slug });

useHead({
  title: product.name,
  meta: [{ hid: 'description', name: 'description', content: product.description }],
});

const quantity = ref(1);
const activeVariation = ref(null);
const variation = ref([]);
const indexOfTypeAny = [];
const attrValues = ref([]);

const type = computed(() => (activeVariation.value ? activeVariation.value : product));
const primaryCategory = computed(() => {
  return product.productCategories?.nodes[0];
});
const selectProductInput = computed(() => ({
  productId: type.value.databaseId,
  quantity: quantity.value,
}));

const checkForVariationTypeOfAny = () => {
  const numberOfVariation = product.attributes.nodes.length;
  for (let index = 0; index < numberOfVariation; index++) {
    const tempArray = [];
    product.variations.nodes.forEach((element) => {
      if (element.attributes.nodes[index]?.value) tempArray.push(element.attributes.nodes[index].value);
    });

    if (!tempArray.some(Boolean)) indexOfTypeAny.push(index);
  }
};

onMounted(() => {
  if (product.variations) checkForVariationTypeOfAny(product.variations.nodes);
});

const updateSelectedVariations = (variations) => {
  if (!product.variations) {
    return;
  }

  attrValues.value = variations.map((el) => ({
    attributeName: el.name,
    attributeValue: el.value,
  }));
  const cloneArray = JSON.parse(JSON.stringify(variations));
  const getActiveVariation = product.variations.nodes.filter((variation) => {
    // If there is any variation of type ANY set the value to ''
    indexOfTypeAny.forEach((index) => (cloneArray[index].value = ''));
    return arraysEqual(formatArray(variation.attributes.nodes), formatArray(cloneArray));
  });

  activeVariation.value = getActiveVariation[0];
  selectProductInput.value.variationId = activeVariation.value?.databaseId || null;
  selectProductInput.value.variation = activeVariation.value ? attrValues.value : null;
  variation.value = variations;
};
</script>

<style>
.product-categories > a:last-child .comma {
  display: none;
}

input[type='number']::-webkit-inner-spin-button {
  opacity: 1;
}
</style>
