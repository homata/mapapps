OSMJa handon


Ruby on Railsの話

必要なモノ

ruby (1.9.3推奨)
http://www.ruby-lang.org/ja/downloads/
leaflet.js 0.5.1
http://leafletjs.com/download.html

railsインストール

$ gem install rails
$ rails new mapapp
$ cd mapapp/

起動確認
$ bundle exec rails s

hamlを導入
参考資料: マークアッパー的 Haml入門21の手引き http://fukuyama.co/haml2

$ emacs Gemfile

gem 'haml'
gem 'haml-rails'

$ bundle install

$ mv app/views/layouts/application.html.erb app/views/layouts/application.html.haml
$ emacs app/views/layouts/application.html.haml
!!!
%html
  %head
    %title Mapapp
    = stylesheet_link_tag "application", :media => "all"
    = javascript_include_tag "application"
    = csrf_meta_tags
%body
  = yield

scaffold

$ bundle exec rails generate scaffold map title:string latitude:float longitude:float

$ bundle exec rake db:migrate

Leaflet.jsをコピー

$ cp ~/Downloads/Leaflet/dist/leaflet.js vendor/assets/javascripts/
$ cp ~/Downloads/Leaflet/dist/leaflet.css vendor/assets/stylesheets/
$ cp ~/Downloads/Leaflet/dist/leaflet.ie.css vendor/assets/stylesheets/
$ mkdir vendor/assets/images
$ cp ~/Downloads/Leaflet/dist/images/* vendor/assets/images/
$ bundle exec rails s

application.js

$ emacs app/assets/javascripts/application.js
...
//= require jquery
//= require jquery_ujs
//= require leaflet
//= require_tree .

application.css

$ emacs app/assets/stylesheets/application.css
...
 *= require_self
 *= require leaflet
 *= require_tree .
 */

application.html.haml

$ emacs app/views/layouts/application.html.haml
...
    %title Mapapp
    = stylesheet_link_tag "application", :media => "all"

    /[if lte IE 8]
      = stylesheet_link_tag "leaflet.ie"
    = javascript_include_tag "application"
    = csrf_meta_tags
...

_form.html.haml(1)

$ emacs app/views/maps/_form.html.haml
...
%div#map
:css
  #map { height: 400px; }
:javascript
  L.Icon.Default.imagePath = "/assets";
  var map = L.map('map').setView([35.64483, 139.40881], 17);
  var osmUrl = "http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png";
  var osmAttrib = "Map data © OpenStreetMap contributors";
  L.tileLayer(osmUrl, {
    attribution: osmAttrib,
    maxZoom: 20,
  }).addTo(map);

_form.html.haml(2)

$ emacs app/views/maps/_form.html.haml
...(続き)
  var marker = L.marker([35.64483, 139.40881]).addTo(map);
  map.on("click", function(e) {
    marker.setLatLng(e.latlng);
    $("#map_latitude").val(e.latlng.lat);
    $("#map_longitude").val(e.latlng.lng);
  });

_form.html.haml(3)

$ emacs app/views/maps/_form.html.haml
...
:javascript
  L.Icon.Default.imagePath = "/assets";
  var lat = $("#map_latitude").val()? $("#map_latitude").val() : 35.64483;
  var lng = $("#map_longitude").val()? $("#map_longitude").val() : 139.40881;
  var map = L.map('map').setView([lat, lng], 17);
  var osmUrl = "http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png";

...

  }).addTo(map);
  var marker = L.marker([lat, lng]).addTo(map);
  map.on("click", function(e) {
...

show.html.haml

$ emacs app/views/maps/show.html.haml
...
%div#map
:css
  #map { height: 400px; }
:javascript
  L.Icon.Default.imagePath = "/assets";
  var lat = #{@map.latitude}
  var lng = #{@map.longitude}
  var map = L.map('map').setView([lat, lng], 17);
  var osmUrl = "http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png";
  var osmAttrib = "Map data © OpenStreetMap contributors";
  L.tileLayer(osmUrl, {
    attribution: osmAttrib,
    maxZoom: 20,
  }).addTo(map);
  var marker = L.marker([lat, lng]).addTo(map);

index.html(1)

$ emacs app/views/maps/index.html.haml
...
  L.Icon.Default.imagePath = "/assets";
  var lat = 35.64483;
  var lng = 139.40881;
  var map = L.map('map').setView([lat, lng], 17);
  var osmUrl = "http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png";
  var osmAttrib = "Map data © OpenStreetMap contributors";
  L.tileLayer(osmUrl, {
    attribution: osmAttrib,
    maxZoom: 20,
  }).addTo(map);

index.html(2)

$ emacs app/views/maps/index.html.haml
...
  $.ajax({
    type: "GET",
    url: "/maps.json",
    dataType: "json",
    success: function(maps) {
      for(var i = 0; i < maps.length; i++) {
        var m = maps[i];
        var marker = L.marker([m.latitude, m.longitude]).addTo(map);
        marker.bindPopup(m.title);
      }
    }
  });


こっからAndroidの話


必要なファイル

osmdroid-android-3.0.8.jar
http://code.google.com/p/osmdroid/downloads/list
slf4j-android-1.5.8.jar
http://www.slf4j.org/android/


Manifest.xml

<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-feature android:name="android.hardware.location.network" />
<uses-feature android:name="android.hardware.location.gps" />
<uses-feature android:name="android.hardware.wifi" />

MainActivity.java(1)

import org.osmdroid.util.GeoPoint;
import org.osmdroid.views.MapView;

public class MainActivity extends Activity {

    private MapView mapView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.mapView = new MapView(this, 256);
        this.mapView.setClickable(true);
        this.mapView.setBuiltInZoomControls(true);

        this.mapView.getController().setZoom(17);
        this.mapView.getController().setCenter(new GeoPoint(35.64483, 139.40881));

        setContentView(mapView);
    }

MainActivity.java(2)

import org.osmdroid.DefaultResourceProxyImpl;
import org.osmdroid.ResourceProxy;
import org.osmdroid.views.overlay.ItemizedIconOverlay;
import org.osmdroid.views.overlay.OverlayItem;

    private ItemizedIconOverlay<OverlayItem> myLocationOverlay;


        GeoPoint overlayPoint = new GeoPoint(35.64483, 139.40881);
        ArrayList<OverlayItem> overlays = new ArrayList<OverlayItem>();
        overlays.add(new OverlayItem("明星大学", "OSCの会場だよ", overlayPoint));
        ResourceProxy resourceProxy = new DefaultResourceProxyImpl(getApplicationContext());
        this.myLocationOverlay = new ItemizedIconOverlay<OverlayItem>(overlays, null, resourceProxy);
        this.mapView.getOverlays().add(this.myLocationOverlay);
        this.mapView.invalidate();


MainActivity.java(3)

    @SuppressLint("ShowToast")
    public boolean onSingleTapUpHelper(int i, OverlayItem item) {
        StringBuffer bf = new StringBuffer();
        bf.append("Tap: ").append(item.mTitle).append(":").append(item.mDescription);
        Toast.makeText(getApplicationContext(), bf.toString(), Toast.LENGTH_LONG).show();
        return true;
    }

MainActivity.java(4)

       OnItemGestureListener<OverlayItem> gesture = new ItemizedIconOverlay.OnItemGestureListener<OverlayItem>() {
            @Override
            public boolean onItemLongPress(int arg0, OverlayItem arg1) {
                return true;
            }
            @Override
            public boolean onItemSingleTapUp(int arg0, OverlayItem arg1) {
                return onSingleTapUpHelper(arg0, arg1);
            }
        };
        this.myLocationOverlay = new ItemizedIconOverlay<OverlayItem>(overlays, gesture, resourceProxy);


