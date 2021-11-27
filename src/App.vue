<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width,initial-scale=1.0" />
    <link rel="icon" href="<%= BASE_URL %>favicon.ico" />
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
    <script src="https://maps.googleapis.com/maps/api/js?key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX&callback=initMap&libraries=places"></script>
  </body>
</html>

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
  name: "app",
  data: () => ({
    map: null,
    distance: 0,
    now_lat: 0,
    now_lng: 0,
    data: [],
    List: [],
    res: "",
    id_List_01: [],
    id_List_02: [],
    id_List_03: [],
    id_List_04: [],
    id_List_05: [],
  }),
  methods: {
    initMap: function() {
      if (navigator.geolocation) {
        // callback関数内でthis使えないため
        let vm = this;
        navigator.geolocation.getCurrentPosition(function(position) {
          vm.now_lat = position.coords.latitude;
          vm.now_lng = position.coords.longitude;

          vm.map = new window.google.maps.Map(document.getElementById("map"), {
            center: new window.google.maps.LatLng(vm.now_lat, vm.now_lng),
            zoom: 15,
            mapTypeId: window.google.maps.MapTypeId.ROADMAP,
          });

          // 読み込み中表示
          vm.res = document.getElementById("results");
          vm.res.innerHTML = "Now Loading...";

          // 検索範囲を指定
          vm.distance = document.getElementById("distance").value;

          let request = {
            location: new window.google.maps.LatLng(vm.now_lat, vm.now_lng),
            radius: vm.distance, // ※１ 表示する半径領域を設定(1 = 1M)
            types: ["restaurant", "cafe"], // ※２ typesプロパティの施設タイプを設定
            map: vm.map,
          };

          let service = new window.google.maps.places.PlacesService(vm.map);
          service.nearbySearch(request, Result_Places);

          function Result_Places(results, status, pagination) {
            if (status == window.google.maps.places.PlacesServiceStatus.OK) {
              if (pagination.hasNextPage) {
                // pagination.nextPageで次の検索結果を表示する。※連続実行すると取得に失敗するので、1秒くらい間隔をおく
                // 検索結果をList配列に連結
                setTimeout(pagination.nextPage(), 1000);
              } else {
                //pagination.hasNextPage==falseになったら、全ての情報が取得できているので、結果を表示する
                vm.List = vm.List.concat(results);
                console.log(vm.List.length);
                DataSet(vm.List, vm.now_lat, vm.now_lng, vm.res);
              }
            } else {
              // エラー表示
              if (
                status ==
                window.google.maps.places.PlacesServiceStatus.ZERO_RESULTS
              ) {
                vm.res.innerHTML = "検索結果が0件です。";
              } else if (
                status == window.google.maps.places.PlacesServiceStatus.ERROR
              ) {
                vm.res.innerHTML = "サーバ接続に失敗しました。";
              } else if (
                status ==
                window.google.maps.places.PlacesServiceStatus.INVALID_REQUEST
              ) {
                vm.res.innerHTML = "リクエストが無効でした。";
              } else if (
                status ==
                window.google.maps.places.PlacesServiceStatus.OVER_QUERY_LIMIT
              ) {
                vm.res.innerHTML = "リクエストの利用制限回数を超えました。";
              } else if (
                status ==
                window.google.maps.places.PlacesServiceStatus.REQUEST_DENIED
              ) {
                vm.res.innerHTML = "サービスが使えない状態でした。";
              } else if (
                status ==
                window.google.maps.places.PlacesServiceStatus.UNKNOWN_ERROR
              ) {
                vm.res.innerHTML = "原因不明のエラーが発生しました。";
              }
            }
          }

          // 距離順に並び変え & 結果を表示
          function DataSet(List, now_lat, now_lng) {
            // 現在地からの距離順でソート（連想配列ソート）
            List.sort(function(a, b) {
              if (
                getDistance(
                  a.geometry.location.lat(),
                  a.geometry.location.lng(),
                  now_lat,
                  now_lng,
                  0
                ) <
                getDistance(
                  b.geometry.location.lat(),
                  b.geometry.location.lng(),
                  now_lat,
                  now_lng,
                  0
                )
              )
                return -1;
              if (
                getDistance(
                  a.geometry.location.lat(),
                  a.geometry.location.lng(),
                  now_lat,
                  now_lng,
                  0
                ) >
                getDistance(
                  b.geometry.location.lat(),
                  b.geometry.location.lng(),
                  now_lat,
                  now_lng,
                  0
                )
              )
                return 1;
              return 0;
            });

            // 配列dataに各距離を記載
            for (let i = 0; i < List.length; i++) {
              vm.data[i] = getDistance(
                List[i].geometry.location.lat(),
                List[i].geometry.location.lng(),
                now_lat,
                now_lng,
                0
              );
            }

            console.log(vm.data);

            // HTML文の記載
            let resultHTML = "<ol>";

            for (let i = 0; i < List.length; i++) {
              let ListData = List[i];
              let dis_data = vm.data[i];
              if (dis_data == undefined) dis_data = "---";
              // 表示内容（名称：距離）
              vm.id_List_01[i] = "List_" + String(i + 1);
              vm.id_List_02[i] = ListData.name;
              vm.id_List_03[i] = ListData.vicinity;
              vm.id_List_04[i] = ListData.geometry.location.lat();
              vm.id_List_05[i] = ListData.geometry.location.lng();

              console.log(vm.id_List_01[i]);
              console.log(ListData.name);
              let content = ListData.name + ": " + dis_data + "m";

              //クリック時にMapにマーカー表示するようにAタグを作成
              resultHTML += "<li>";
              resultHTML += '<a href="javascript: void(0);"';
              resultHTML += "id = " + vm.id_List_01[i] + ">";
              resultHTML += content;
              resultHTML += "</a>";
              resultHTML += "</li>";
            }
            resultHTML += "</ol>";

            // 結果表示
            vm.res.innerHTML = resultHTML;

            for (let r = 0; r < vm.id_List_01.length; r++) {
              let target_id = document.getElementById(vm.id_List_01[r]);
              let array = [
                vm.id_List_02[r],
                vm.id_List_03[r],
                vm.id_List_04[r],
                vm.id_List_05[r],
              ];
              console.log(array);
              target_id.addEventListener("click", function() {
                createMarker(array);
              });
            }
          }

          // マーカー表示する位置のMap表示
          function createMarker(array) {
            console.log(array[0]);
            console.log(array[1]);

            vm.map = new window.google.maps.Map(
              document.getElementById("map"),
              {
                center: new window.google.maps.LatLng(array[2], array[3]),
                zoom: 15,
                mapTypeId: window.google.maps.MapTypeId.ROADMAP,
              }
            );

            // マーカー表示
            let marker = new window.google.maps.Marker({
              map: vm.map,
              position: new window.google.maps.LatLng(array[2], array[3]),
            });

            // 情報窓の設定
            let info = '<div style="min-width: 100px">';
            info += array[0] + "<br />";
            info += array[1] + "<br />";
            info +=
              '<a href="https://maps.google.co.jp/maps?q=' +
              encodeURIComponent(array[0] + " " + array[1]) +
              '&z=15&iwloc=A"';
            info += ' target="_blank">⇒詳細表示</a><br />';
            info +=
              '<a href="https://www.google.com/maps/dir/?api=1&destination=' +
              array[2] +
              "," +
              array[3] +
              '"';
            info += ' target="_blank">⇒ここへ行く</a>';
            info += "</div>";

            // 情報窓の表示
            var infoWindow = new window.google.maps.InfoWindow({
              content: info,
            });
            infoWindow.open(vm.map, marker);

            //マーカーのクリック時にも情報窓を表示する
            marker.addListener("click", function() {
              infoWindow.open(vm.map, marker);
            });
          }

          // マーカーを生成する関数（引数には検索結果の配列 results[i] が入ってきます）
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
  mounted: function() {
    this.initMap();
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
