    # Markie-CAD-<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>VCFD Live CAD</title>

<style>
body{
    margin:0;
    background:#000;
    color:white;
    font-family:Arial,Helvetica,sans-serif;
}

#header{
    background:#111;
    padding:12px;
    font-size:24px;
    font-weight:bold;
}

#updated{
    float:right;
    font-size:14px;
    color:#aaa;
}

#container{
    display:flex;
    height:calc(100vh - 55px);
}

#calls{
    width:40%;
    overflow-y:auto;
    border-right:1px solid #333;
}

#map{
    width:60%;
    height:100%;
}

.call{
    padding:10px;
    border-bottom:1px solid #333;
}

.incident{
    font-size:18px;
    font-weight:bold;
}

.type{
    color:#66ccff;
}

.enroute{
    color:#FFD700;
    font-weight:bold;
}

.onscene{
    color:#ff4444;
    font-weight:bold;
}

.units{
    color:#88ff88;
}
</style>

<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY"></script>
</head>

<body>

<div id="header">
Ventura County Fire CAD
<span id="updated"></span>
</div>

<div id="container">
<div id="calls"></div>
<div id="map"></div>
</div>

<script>

const API =
'https://firefeeds.venturacounty.gov/api/incidents';

let map;
let markers = [];

function initMap(){

    map = new google.maps.Map(
        document.getElementById('map'),
        {
            center:{
                lat:34.28,
                lng:-119.20
            },
            zoom:10,
            mapTypeId:'roadmap'
        }
    );
}

function clearMarkers(){

    markers.forEach(m => m.setMap(null));

    markers = [];
}

async function loadCalls(){

    try{

        const response = await fetch(
            API + '?t=' + Date.now(),
            {
                cache:'no-store'
            }
        );

        const data =
            await response.json();

        clearMarkers();

        data.sort((a,b)=>
            b.incidentNumber.localeCompare(
                a.incidentNumber
            )
        );

        let html='';

        data.forEach(call=>{

            let statusClass='';

            if(
                (call.status || '')
                .toLowerCase()
                .includes('route')
            ){
                statusClass='enroute';
            }

            if(
                (call.status || '')
                .toLowerCase()
                .includes('scene')
            ){
                statusClass='onscene';
            }

            html += `
            <div class="call">

                <div class="incident">
                    ${call.incidentNumber}
                </div>

                <div>
                    ${call.block}
                    ${call.address}
                </div>

                <div>
                    ${call.city}
                </div>

                <div class="type">
                    ${call.incidentType || 'Unknown'}
                </div>

                <div class="${statusClass}">
                    ${call.status || ''}
                </div>

                <div class="units">
                    ${call.units || ''}
                </div>

                <div>
                    ${call.responseDate}
                </div>

            </div>
            `;

            if(
                call.latitude &&
                call.longitude
            ){

                const marker =
                new google.maps.Marker({

                    position:{
                        lat:parseFloat(call.latitude),
                        lng:parseFloat(call.longitude)
                    },

                    map:map,

                    icon:{
                        url:
'https://maps.google.com/mapfiles/kml/paddle/red-blank.png',
                        scaledSize:
                        new google.maps.Size(
                            32,
                            32
                        )
                    },

                    title:
                        call.incidentNumber
                });

                markers.push(marker);
            }

        });

        document.getElementById(
            'calls'
        ).innerHTML = html;

        document.getElementById(
            'updated'
        ).innerHTML =
        'Updated: ' +
        new Date()
        .toLocaleTimeString();

    }
    catch(error){

        console.error(error);

    }

}

initMap();

loadCalls();

setInterval(
    loadCalls,
    15000
);

</script>

</body>
</html>