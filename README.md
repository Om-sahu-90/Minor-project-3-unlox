<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather & Movie Hub</title>
    <style>
        /* --- GENERAL STYLING --- */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background: #f0f2f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            min-height: 100vh;
            padding: 20px 10px;
        }

        .container {
            width: 100%;
            max-width: 480px;
            display: flex;
            flex-direction: column;
            gap: 25px;
        }

        /* --- CARD STYLE FOR BOTH APPS --- */
        .app-card {
            background: #ffffff;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
            text-align: center;
        }

        h2 {
            margin-bottom: 15px;
            color: #333;
            font-size: 22px;
        }

        .search-box {
            display: flex;
            gap: 10px;
            margin-bottom: 15px;
            flex-wrap: wrap;
        }

        input {
            flex: 1;
            min-width: 180px;
            padding: 12px;
            border: 1px solid #ccd1d9;
            border-radius: 8px;
            font-size: 16px;
            outline: none;
        }

        input:focus {
            border-color: #007bff;
        }

        button {
            flex: 1;
            padding: 12px 20px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            font-weight: 600;
            transition: background 0.2s;
        }

        button:hover {
            background: #0056b3;
        }

        /* --- RESULTS LAYOUT --- */
        .result-box {
            margin-top: 15px;
            padding: 15px;
            background: #f8f9fa;
            border-radius: 10px;
            text-align: left;
        }

        .hidden {
            display: none !important;
        }

        .error-msg {
            color: #dc3545;
            margin-top: 12px;
            font-weight: 500;
            font-size: 14px;
        }

        /* --- SPECIFIC MOVIE UI LAYOUT --- */
        .movie-content {
            display: flex;
            flex-direction: column;
            gap: 15px;
            align-items: center;
        }

        .movie-poster {
            width: 100%;
            max-width: 180px;
            border-radius: 6px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.15);
        }

        .movie-details {
            width: 100%;
        }

        .movie-details h3 {
            margin-bottom: 8px;
            color: #111;
        }

        .movie-details p {
            margin-bottom: 5px;
            font-size: 14px;
            color: #555;
        }
    </style>
</head>
<body>

    <main class="container">
        
        <section class="app-card">
            <h2>Weather App</h2>
            <div class="search-box">
                <input type="text" id="weather-input" placeholder="Enter city (e.g., Hinjilicut)...">
                <button id="weather-btn">Search</button>
            </div>
            
            <div id="weather-result" class="result-box hidden" style="text-align: center;">
                <h3 id="city-name" style="margin-bottom: 5px;"></h3>
                <p id="temperature" style="font-size: 28px; font-weight: bold; color: #007bff; margin: 5px 0;"></p>
                <p id="humidity"></p>
                <p id="condition" style="font-weight: 500; text-transform: capitalize;"></p>
            </div>
            <p id="weather-error" class="error-msg hidden"></p>
        </section>


        <section class="app-card">
            <h2>Movie Search App</h2>
            <div class="search-box">
                <input type="text" id="movie-input" placeholder="Enter movie title...">
                <button id="movie-btn">Search</button>
            </div>
            
            <div id="movie-result" class="result-box hidden">
                <div class="movie-content">
                    <img id="movie-poster" class="movie-poster" src="" alt="Movie Poster">
                    <div class="movie-details">
                        <h3 id="movie-title"></h3>
                        <p id="movie-year"></p>
                        <p id="movie-genre"></p>
                        <p id="movie-plot" style="margin-top: 8px; color: #333; line-height: 1.4;"></p>
                    </div>
                </div>
            </div>
            <p id="movie-error" class="error-msg hidden"></p>
        </section>

    </main>

    <script>
        // ==========================================
        // CONFIGURATION & API KEYS
        // ==========================================
        const WEATHER_API_KEY = '2002b5f17497c72577c1e594a4096433'; 
        const MOVIE_API_KEY = 'a54e24e0'; 

        // ==========================================
        // WEATHER APP CONTROLLER
        // ==========================================
        const weatherBtn = document.getElementById('weather-btn');
        const weatherInput = document.getElementById('weather-input');
        const weatherError = document.getElementById('weather-error');
        const weatherResult = document.getElementById('weather-result');

        weatherBtn.addEventListener('click', handleWeatherSearch);
        weatherInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') handleWeatherSearch(); });

        async function handleWeatherSearch() {
            const city = weatherInput.value.trim();
            if (!city) {
                showWeatherError('Please enter a city name.');
                return;
            }
            
            try {
                const url = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${WEATHER_API_KEY}&units=metric`;
                const response = await fetch(url);
                
                if (!response.ok) {
                    throw new Error('City not found. Try a different city name.');
                }
                
                const data = await response.json();
                displayWeather(data);
            } catch (error) {
                showWeatherError(error.message);
            }
        }

        function displayWeather(data) {
            weatherError.classList.add('hidden');
            weatherResult.classList.remove('hidden');
            
            // ES6 Destructuring
            const { name, main: { temp, humidity }, weather, sys: { country } } = data;
            const [{ description }] = weather;
            
            // Template Literals
            document.getElementById('city-name').textContent = `${name}, ${country}`;
            document.getElementById('temperature').textContent = `${Math.round(temp)}°C`;
            document.getElementById('humidity').textContent = `Humidity: ${humidity}%`;
            document.getElementById('condition').textContent = description;
        }

        function showWeatherError(message) {
            weatherResult.classList.add('hidden');
            weatherError.textContent = message;
            weatherError.classList.remove('hidden');
        }


        // ==========================================
        // MOVIE SEARCH APP CONTROLLER
        // ==========================================
        const movieBtn = document.getElementById('movie-btn');
        const movieInput = document.getElementById('movie-input');
        const movieError = document.getElementById('movie-error');
        const movieResult = document.getElementById('movie-result');

        movieBtn.addEventListener('click', handleMovieSearch);
        movieInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') handleMovieSearch(); });

        async function handleMovieSearch() {
            const title = movieInput.value.trim();
            if (!title) {
                showMovieError('Please enter a movie title.');
                return;
            }

            try {
                const url = `https://www.omdbapi.com/?t=${encodeURIComponent(title)}&apikey=${MOVIE_API_KEY}`;
                const response = await fetch(url);
                
                if (!response.ok) {
                    throw new Error('Movie API connection error.');
                }
                
                const data = await response.json();
                
                if (data.Response === "False") {
                    throw new Error(data.Error || 'Movie not found!');
                }
                
                displayMovie(data);
            } catch (error) {
                showMovieError(error.message);
            }
        }

        function displayMovie(data) {
            movieError.classList.add('hidden');
            movieResult.classList.remove('hidden');
            
            // ES6 Destructuring
            const { Title, Year, Genre, Plot, Poster } = data;
            
            // Dynamic content rendering using template literals
            document.getElementById('movie-title').textContent = Title;
            document.getElementById('movie-year').textContent = `Year: ${Year || 'N/A'}`;
            document.getElementById('movie-genre').textContent = `Genre: ${Genre || 'N/A'}`;
            document.getElementById('movie-plot').textContent = Plot || 'No summary description available.';
            
            const posterImg = document.getElementById('movie-poster');
            if (Poster && Poster !== "N/A") {
                posterImg.src = Poster;
                posterImg.classList.remove('hidden');
            } else {
                posterImg.classList.add('hidden');
            }
        }

        function showMovieError(message) {
            movieResult.classList.add('hidden');
            movieError.textContent = message;
            movieError.classList.remove('hidden');
        }
    </script>
</body>
</html>
