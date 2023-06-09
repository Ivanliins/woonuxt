<script setup>
const route = useRoute();
const props = defineProps({
  node: { type: Object, default: null },
  index: { type: Number, default: 1 },
});

const imgWidth = 400;
const imgHeight = 500;

// example: ?filter=pa_color[green,blue],pa_size[large]
const filterQuery = ref(route.query.filter);
const paColor = ref(filterQuery.value?.split('pa_color[')[1]?.split(']')[0]?.split(',') || []);

// watch filterQuery
watch(
  () => route.query,
  () => {
    filterQuery.value = route.query.filter;
    paColor.value = filterQuery.value?.split('pa_color[')[1]?.split(']')[0]?.split(',') || [];
  }
);

const imageSrc = computed(() => {
  const fallbackImage = '/images/placeholder.jpg';
  // If we have a color filter, we need to find the image that matches the color
  if (paColor.value.length) {
    const activeColorImage = props.node?.variations?.nodes.filter((variation) => paColor.value.some((color) => variation.slug.includes(color)));
    if (activeColorImage && activeColorImage.length) {
      return activeColorImage[0].image?.sourceUrl || fallbackImage;
    }
  }
  return props.node?.image?.sourceUrl || fallbackImage;
});
</script>

<template>
  <NuxtLink :to="`/product/${node.slug}`" class="relative product-card">
    <SaleBadge :node="node" class="absolute top-2 right-2" />
    <NuxtImg
      v-if="imageSrc"
      :width="imgWidth"
      :height="imgHeight"
      :src="imageSrc"
      :alt="node.image?.altText || node.name"
      :title="node.image?.title || node.name"
      :loading="index <= 1 ? 'eager' : 'lazy'"
      format="webp" />

    <div class="p-2">
      <StarRating :rating="node.averageRating" :count="node.reviewCount" />
      <h2 class="mb-2 font-light leading-tight">{{ node.name }}</h2>
      <ProductPrice class="text-sm" :sale-price="node.salePrice" :regular-price="node.regularPrice" />
    </div>
  </NuxtLink>
</template>

<style lang="postcss">
.product-card img {
  @apply rounded-lg object-top object-cover w-full;
  aspect-ratio: 1/1.125;
}
</style>
