<script>
  import { onMount } from "svelte";

  let city = ""; // 주소에서 가져온 지역 값
  let currentPage = 1;
  let itemsPerPage = 10;

  let category = "food"; // 기본 카테고리
  let currentFoods = [];
  let totalPages = 0;

  function changePage(page) {
    if (page < 1 || page > totalPages) return;
    currentPage = page;
    fetchPlaces();
  }

  // 서버에서 데이터를 가져오는 함수
  const fetchPlaces = async () => {
    try {
      const params = new URLSearchParams({
        name: city || "", // 'name' 파라미터에 지역명 전달
        page: currentPage,
        limit: itemsPerPage,
      });

      const response = await fetch(
        `http://localhost:8080/api/${category}?${params.toString()}`
      );
      if (!response.ok) throw new Error("네트워크 응답에 문제가 있습니다.");
      const data = await response.json();

      currentFoods = data.places; // 서버에서 받은 데이터
      totalPages = Math.ceil(data.totalCount / itemsPerPage); // 전체 페이지 수 계산
    } catch (error) {
      console.error("데이터를 가져오는 중 오류가 발생했습니다:", error);
    }
  };

  // 전체목록보기를 클릭했을 때 모든 항목을 로드하는 함수
  function loadAllFoods() {
    itemsPerPage = data.totalCount || 1000; // 충분히 큰 수로 설정하거나 서버에서 totalCount를 받아와 설정
    currentPage = 1;
    fetchPlaces();
  }

  onMount(() => {
    const params = new URLSearchParams(window.location.search);
    city = params.get("name") || ""; // URL에서 지역명 가져오기
    fetchPlaces(); // 데이터 가져오기
  });
</script>

<body>
  <div class="PlacePage">
    <h1>{city} 음식점</h1>
    <ul>
      {#each currentFoods as food}
        <a href="" target="_blank">
          <li>
            <p>{food.name}</p>
            {food.address}
          </li>
        </a>
      {/each}
    </ul>
    <div class="pagination">
      <button
        class={currentPage > 1 ? "" : "hidden"}
        on:click={() => changePage(currentPage - 1)}>&lt;</button
      >
      <p>{currentPage}/{totalPages}</p>
      <button
        class={currentPage < totalPages ? "" : "hidden"}
        on:click={() => changePage(currentPage + 1)}>&gt;</button
      >
    </div>
  </div>
</body>
