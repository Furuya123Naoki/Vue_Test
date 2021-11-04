<template>
  <div id="app">
    <select id="distance" v-on:change="initMap()">
      <option value="500"
        >検索範囲を選択してください・・・（デフォルト：500m）</option
      >
      <option value="100">100m</option>
      <option value="200">200m</option>
      <option value="300">300m</option>
      <option value="500">500m</option>
      <option value="1000">1km</option>
      <option value="2000">2km</option>
    </select>
    <div id="map" ref="map"></div>
    <div
      id="results"
      style="width: 100%; height: 200px; border: 1px dotted; padding: 10px; overflow-y: scroll; "
    ></div>
  </div>
</template>

<script>
export default {
  name: "App",
  data: () => ({
    map: null,
    distance: 0,
    now_lat: 0,
    now_lng: 0,
    data: [],
    res: "",
  }),
  mounted() {
    if (navigator.geolocation) {
      // callback関数内でthis使えないため
      let vm = this;
      navigator.geolocation.getCurrentPosition(function(position) {
        vm.now_lat = position.coords.latitude;
        vm.now_lng = position.coords.longitude;

        console.log(vm.now_lat);
        console.log(vm.now_lng);

        let latlng = new window.google.maps.LatLng(vm.now_lat, vm.now_lng);

        vm.map = new window.google.maps.Map(vm.$refs["map"], {
          center: latlng,
          zoom: 16,
        });

        // vm.res = document.getElementById("results").innerHTML;

        vm.distance = document.getElementById("distance").value;

        let request = {
          location: latlng,
          radius: vm.distance, // ※１ 表示する半径領域を設定(1 = 1M)
          types: ["restaurant", "cafe"], // ※２ typesプロパティの施設タイプを設定
          map: vm.map,
        };

        let service = new window.google.maps.places.PlacesService(vm.map);
        service.nearbySearch(request, Result_Places);

        function Result_Places(results, status, pagination) {
          console.log(results);
          if (status == window.google.maps.places.PlacesServiceStatus.OK) {
            if (pagination.hasNextPage) {
              //pagination.nextPageで次の検索結果を表示する
              //※連続実行すると取得に失敗するので、
              //  1秒くらい間隔をおく
              //results の数だけ for 文で繰り返し処理
              for (let i = 0; i < results.length; i++) {
                console.log(results.length);
                // 各距離を計測
                vm.data[i] = getDistance(
                  results[i].geometry.location.lat(),
                  results[i].geometry.location.lng(),
                  vm.now_lat,
                  vm.now_lng,
                  0
                );
                //createMarker() はマーカーを生成する関数（別途定義）
                createMarker(results[i], vm.data[i]);
              }
              setTimeout(pagination.nextPage(), 1000);

              //pagination.hasNextPage==falseになったら
              //全ての情報が取得できているので、
              //結果を表示する
            } else {
              //results の数だけ for 文で繰り返し処理
              for (let i = 0; i < results.length; i++) {
                console.log(results.length);
                // 各距離を計測
                vm.data[i] = getDistance(
                  results[i].geometry.location.lat(),
                  results[i].geometry.location.lng(),
                  vm.now_lat,
                  vm.now_lng,
                  0
                );
                //createMarker() はマーカーを生成する関数（別途定義）
                createMarker(results[i], vm.data[i]);
              }
            }
          }
        }

        function createMarker(place, dis) {
          let marker = new window.google.maps.Marker({
            position: place.geometry.location,
            map: vm.map,
          });

          // マーカーのwindow
          var infowindow = new window.google.maps.InfoWindow({
            content: "店名：" + place.name + "<br>距離：" + dis + "m",
          });

          // マーカークリック時にwindow表示
          marker.addListener("click", function() {
            infowindow.open(vm.map, marker);
          });
          // this.formattedMarkers.push(marker);
        }

        //マーカーを生成する関数（引数には検索結果の配列 results[i] が入ってきます）
        function getDistance(lat1, lng1, lat2, lng2, precision) {
          var dist = 0;
          if (
            Math.abs(lat1 - lat2) < 0.00001 &&
            Math.abs(lng1 - lng2) < 0.00001
          ) {
            dist = 0;
          } else {
            lat1 = (lat1 * Math.PI) / 180;
            lng1 = (lng1 * Math.PI) / 180;
            lat2 = (lat2 * Math.PI) / 180;
            lng2 = (lng2 * Math.PI) / 180;
            var A = 6378140;
            var B = 6356755;
            var F = (A - B) / A;
            var P1 = Math.atan((B / A) * Math.tan(lat1));
            var P2 = Math.atan((B / A) * Math.tan(lat2));
            var X = Math.acos(
              Math.sin(P1) * Math.sin(P2) +
                Math.cos(P1) * Math.cos(P2) * Math.cos(lng1 - lng2)
            );
            var L =
              (F / 8) *
              (((Math.sin(X) - X) * Math.pow(Math.sin(P1) + Math.sin(P2), 2)) /
                Math.pow(Math.cos(X / 2), 2) -
                ((Math.sin(X) - X) * Math.pow(Math.sin(P1) - Math.sin(P2), 2)) /
                  Math.pow(Math.sin(X), 2));
            dist = A * (X + L);
            var decimal_no = Math.pow(10, precision);
            dist = Math.round((decimal_no * dist) / 1) / decimal_no;
          }
          return dist;
        }
      });
    }
  },
  methods: {
    initMap: function() {
      if (navigator.geolocation) {
        // callback関数内でthis使えないため
        let vm = this;
        navigator.geolocation.getCurrentPosition(function(position) {
          vm.now_lat = position.coords.latitude;
          vm.now_lng = position.coords.longitude;

          let latlng = new window.google.maps.LatLng(vm.now_lat, vm.now_lng);

          vm.map = new window.google.maps.Map(vm.$refs["map"], {
            center: latlng,
            zoom: 16,
          });

          vm.distance = document.getElementById("distance").value;

          let request = {
            location: latlng,
            radius: vm.distance, // ※１ 表示する半径領域を設定(1 = 1M)
            types: ["restaurant", "cafe"], // ※２ typesプロパティの施設タイプを設定
            map: vm.map,
          };

          let service = new window.google.maps.places.PlacesService(vm.map);
          service.nearbySearch(request, Result_Places);

          function Result_Places(results, status, pagination) {
            console.log(results);

            if (status == window.google.maps.places.PlacesServiceStatus.OK) {
              if (pagination.hasNextPage) {
                //pagination.nextPageで次の検索結果を表示する
                //※連続実行すると取得に失敗するので、
                //  1秒くらい間隔をおく
                //results の数だけ for 文で繰り返し処理
                for (let i = 0; i < results.length; i++) {
                  console.log(results.length);
                  // 各距離を計測
                  vm.data[i] = getDistance(
                    results[i].geometry.location.lat(),
                    results[i].geometry.location.lng(),
                    vm.now_lat,
                    vm.now_lng,
                    0
                  );
                  //createMarker() はマーカーを生成する関数（別途定義）
                  createMarker(results[i], vm.data[i]);
                }
                setTimeout(pagination.nextPage(), 1000);

                //pagination.hasNextPage==falseになったら
                //全ての情報が取得できているので、
                //結果を表示する
              } else {
                //results の数だけ for 文で繰り返し処理
                for (let i = 0; i < results.length; i++) {
                  console.log(results.length);
                  // 各距離を計測
                  vm.data[i] = getDistance(
                    results[i].geometry.location.lat(),
                    results[i].geometry.location.lng(),
                    vm.now_lat,
                    vm.now_lng,
                    0
                  );
                  //createMarker() はマーカーを生成する関数（別途定義）
                  createMarker(results[i], vm.data[i]);
                }
              }
            }
          }

          // マップの中心位置を指定
          // vm.map.setCenter(vm.now_lat, vm.now_lng);

          //マーカーを生成する関数（引数には検索結果の配列 results[i] が入ってきます）
          function createMarker(place, dis) {
            let marker = new window.google.maps.Marker({
              position: place.geometry.location,
              map: vm.map,
            });

            // マーカーのwindow
            var infowindow = new window.google.maps.InfoWindow({
              content: "店名：" + place.name + "<br>距離：" + dis + "m",
            });

            // マーカークリック時にwindow表示
            marker.addListener("click", function() {
              infowindow.open(vm.map, marker);
            });
            // this.formattedMarkers.push(marker);
          }

          //マーカーを生成する関数（引数には検索結果の配列 results[i] が入ってきます）
          function getDistance(lat1, lng1, lat2, lng2, precision) {
            var dist = 0;
            if (
              Math.abs(lat1 - lat2) < 0.00001 &&
              Math.abs(lng1 - lng2) < 0.00001
            ) {
              dist = 0;
            } else {
              lat1 = (lat1 * Math.PI) / 180;
              lng1 = (lng1 * Math.PI) / 180;
              lat2 = (lat2 * Math.PI) / 180;
              lng2 = (lng2 * Math.PI) / 180;
              var A = 6378140;
              var B = 6356755;
              var F = (A - B) / A;
              var P1 = Math.atan((B / A) * Math.tan(lat1));
              var P2 = Math.atan((B / A) * Math.tan(lat2));
              var X = Math.acos(
                Math.sin(P1) * Math.sin(P2) +
                  Math.cos(P1) * Math.cos(P2) * Math.cos(lng1 - lng2)
              );
              var L =
                (F / 8) *
                (((Math.sin(X) - X) *
                  Math.pow(Math.sin(P1) + Math.sin(P2), 2)) /
                  Math.pow(Math.cos(X / 2), 2) -
                  ((Math.sin(X) - X) *
                    Math.pow(Math.sin(P1) - Math.sin(P2), 2)) /
                    Math.pow(Math.sin(X), 2));
              dist = A * (X + L);
              var decimal_no = Math.pow(10, precision);
              dist = Math.round((decimal_no * dist) / 1) / decimal_no;
            }
            return dist;
          }
        });
      }
    },
  },
};
</script>

<style>
#map {
  height: 400px;
  background: gray;
}

html,
body {
  height: 100%;
  margin: 0;
  padding: 0;
}
</style>
