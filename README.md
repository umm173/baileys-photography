<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bailey's Photography</title>
    <style>
        /* Reset */
        * {margin: 0; padding: 0; box-sizing: border-box;}
        body {font-family: Arial, sans-serif; line-height: 1.6; color: #333; background: #fdfdfd;}

        /* Header */
        header {background: #222; color: white; padding: 20px 0; text-align: center;}
        header h1 {margin-bottom: 10px;}
        nav a {color: white; margin: 0 15px; text-decoration: none; font-weight: bold;}
        nav a:hover {text-decoration: underline;}

        /* Hero Section */
        .hero {text-align: center; padding: 80px 20px; background: url('https://source.unsplash.com/1600x600/?photography') center/cover no-repeat; color: white;}
        .hero h2 {font-size: 3em; margin-bottom: 10px; text-shadow: 2px 2px 5px rgba(0,0,0,0.5);}
        .hero p {font-size: 1.5em; text-shadow: 1px 1px 3px rgba(0,0,0,0.5);}

        /* Sections */
        section {padding: 60px 20px; max-width: 1000px; margin: auto;}
        section h2 {text-align: center; margin-bottom: 40px; font-size: 2.5em;}

        /* About */
        .about p {text-align: center; font-size: 1.2em;}

        /* Gallery */
        .gallery {display: flex; flex-wrap: wrap; justify-content: center;}
        .gallery img {width: 30%; margin: 1.66%; border-radius: 5px; transition: transform 0.3s;}
        .gallery img:hover {transform: scale(1.05);}

        /* Contact */
        .contact-form {max-width: 600px; margin: auto;}
        .contact-form p {margin-bottom: 10px; text-align: center; font-size: 1.1em;}
        form input, form textarea {width: 100%; margin: 5px 0 15px 0; padding: 10px; border: 1px solid #ccc; border-radius: 5px;}
        button {padding: 10px 20px; background: #222; color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 1em;}
        button:hover {background: #555;}

        /* Footer */
        footer {text-align: center; padding: 20px; background: #222; color: white; margin-top: 40px;}

        /* Responsive */
        @media(max-width: 768px){
            .gallery img {width: 45%;}
        }
        @media(max-width: 480px){
            .gallery img {width: 100%;}
        }
    </style>
</head>
<body>

    <!-- Header -->
    <header>
        <h1>Bailey's Photography</h1>
        <nav>
            <a href="#home">Home</a>
            <a href="#portfolio">Portfolio</a>
            <a href="#contact">Contact</a>
        </nav>
    </header>

    <!-- Hero -->
    <section class="hero" id="home">
        <h2>Capturing Moments, Creating Memories</h2>
        <p>Professional photography for any occasion</p>
    </section>

    <!-- About -->
    <section class="about">
        <h2>About Me</h2>
        <p>Hello! I’m Bailey Moore, a professional photographer specializing in landscapes, portraits, and events. My mission is to capture your moments perfectly and turn them into lasting memories.</p>
    </section>

    <!-- Portfolio -->
    <section class="portfolio" id="portfolio">
        <h2>Portfolio</h2>
        <div class="gallery">
            <img src="https://source.unsplash.com/400x300/?landscape" alt="Photo 1">
            <img src="https://source.unsplash.com/400x300/?portrait" alt="Photo 2">
            <img src="https://source.unsplash.com/400x300/?nature" alt="Photo 3">
            <img src="https://source.unsplash.com/400x300/?wedding" alt="Photo 4">
            <img src="https://source.unsplash.com/400x300/?travel" alt="Photo 5">
            <img src="https://source.unsplash.com/400x300/?event" alt="Photo 6">
        </div>
    </section>

    <!-- Contact -->
    <section class="contact" id="contact">
        <h2>Contact Me</h2>
        <div class="contact-form">
            <p>Email: <a href="mailto:bailey@example.com">bailey@example.com</a></p>
            <p>Phone: +64 123 456 789</p>
            <form>
                <input type="text" name="name" placeholder="Your Name" required>
                <input type="email" name="email" placeholder="Your Email" required>
                <textarea name="message" placeholder="Your Message" rows="5" required></textarea>
                <button type="submit">Send Message</button>
            </form>
        </div>
    </section>

    <!-- Footer -->
    <footer>
        <p>&copy; 2025 Bailey's Photography</p>
    </footer>

</body>
</html>
