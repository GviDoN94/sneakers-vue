<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

    <div class="flex gap-4">
      <select
        @change="onChangeSelect"
        class="py-2 px-3 border rounded-md outline-none"
      >
        <option value="name">По названию</option>
        <option value="price">По цене (дешевые)</option>
        <option value="-price">По цене (дорогие)</option>
      </select>

      <div class="relative">
        <img
          class="absolute top-2.5 left-4"
          src="/search.svg"
          alt="Search"
        />
        <input
          @change="onChangeSearchInput"
          type="text"
          placeholder="Поиск..."
          class="border rounded-md py-1.5 pl-11 pr-4 outline-none focus:border-gray-400"
        />
      </div>
    </div>
  </div>

  <div class="mt-10">
    <CardList
      :items="items"
      @add-to-favorite="addToFavorite"
      @add-to-cart="onClickAddPlus"
    />
  </div>
</template>

<script setup>
  import { ref, reactive, inject, onMounted, watch } from 'vue';
  import axios from 'axios';
  import debounce from 'lodash.debounce';

  import CardList from '@/components/CardList.vue';

  const { VITE_BASE_API: baseApi } = import.meta.env;

  const { cart, onClickAddPlus } = inject('cart');

  const items = ref([]);

  const filters = reactive({
    sortBy: 'title',
    searchQuery: '',
  });

  const onChangeSelect = (event) => (filters.sortBy = event.target.value);

  const onChangeSearchInput = debounce(
    (event) => (filters.searchQuery = event.target.value),
    300,
  );

  const fetchFavorites = async () => {
    try {
      const { data: favorites } = await axios.get(`${baseApi}/favorites`);

      items.value = items.value.map((item) => {
        const favorite = favorites.find(
          (favorite) => favorite.item_id === item.id,
        );

        if (!favorite) {
          return item;
        }

        return {
          ...item,
          isFavorite: true,
          favoriteId: favorite.id,
        };
      });
    } catch (e) {
      console.log(e);
    }
  };

  const addToFavorite = async (item) => {
    try {
      if (!item.isFavorite) {
        const obj = {
          item_id: item.id,
        };

        item.isFavorite = true;

        const { data } = await axios.post(`${baseApi}/favorites`, obj);

        item.favoriteId = data.id;
      } else {
        item.isFavorite = false;
        await axios.delete(`${baseApi}/favorites/${item.favoriteId}`);
        item.favoriteId = null;
      }
    } catch (e) {
      console.log(e);
    }
  };

  const fetchItems = async () => {
    try {
      const params = {
        sortBy: filters.sortBy,
      };

      if (filters.searchQuery) {
        params.title = `*${filters.searchQuery}*`;
      }

      const { data } = await axios.get(`${baseApi}/items`, { params });

      items.value = data.map((obj) => ({
        ...obj,
        isFavorite: false,
        favoriteId: null,
        isAdded: false,
      }));
    } catch (e) {
      console.log(e);
    }
  };

  onMounted(async () => {
    await fetchItems();
    await fetchFavorites();

    items.value = items.value.map((item) => ({
      ...item,
      isAdded: cart.value.some((cartItem) => cartItem.id === item.id),
    }));
  });

  watch(cart, () => {
    items.value = items.value.map((item) => ({ ...item, isAdded: false }));
  });

  watch(filters, fetchItems);
</script>
