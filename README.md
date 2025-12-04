# Maintenance Page - TeamLease EdTech

A beautiful, responsive maintenance page with animated effects, glassmorphism design, and floating icons. Perfect for displaying scheduled production deployments.

## Features

- ✨ **Modern UI Design**: Glassmorphism effects with blur and transparency
- 🎨 **Animated Background**: Gradient mesh with smooth animations
- 🖼️ **Side Illustrations**: Large PNG images on left and right sides
- 🎭 **Floating Icons**: Animated wrench, nut, bolt, and setting icons
- 📱 **Fully Responsive**: Optimized for desktop, tablet, and mobile devices
- ⚡ **CSS-Only Animations**: Smooth, performant animations without JavaScript
- 🎯 **Single HTML File**: Self-contained with inline CSS

## Project Structure

```
.
├── index.html          # Main HTML file with all styles and content
├── logo.png           # PNG illustration asset
├── Dockerfile         # Docker container configuration
├── nginx.conf         # Nginx web server configuration
├── README.md          # This file
└── DEPLOYMENT_GUIDE.md # AWS ECS deployment instructions
```

## Quick Start

### Local Development

1. Clone the repository:
```bash
git clone <repository-url>
cd "NEW Maintance Page"
```

2. Open `index.html` in a web browser or use a local server:
```bash
# Using Python
python -m http.server 8000

# Using Node.js (http-server)
npx http-server -p 8000

# Using PHP
php -S localhost:8000
```

3. Open `http://localhost:8000` in your browser

### Docker Local Testing

```bash
# Build the image
docker build -t maintenance-page .

# Run the container
docker run -d -p 8080:80 --name maintenance-page maintenance-page

# Access at http://localhost:8080
```

## Customization

### Changing the Logo/Image

Replace `logo.png` with your own PNG image. The image path is referenced in:
- Line 953: `<img src="logo.png" ...>`
- Line 954: `<img src="logo.png" ...>`

### Modifying Text Content

Edit the content in `index.html`:
- **Heading**: Line 998 - "Scheduled Production Deployment in Progress"
- **Main Content**: Lines 1000-1003
- **ETA Section**: Lines 1010-1011
- **Closing Line**: Line 1016

### Adjusting Colors and Styles

All styles are in the `<style>` block (lines 10-945). Key sections:
- Background gradients: Lines 120-144
- Container styles: Lines 146-171
- Image positioning: Lines 35-58
- Animation keyframes: Lines 60-117

## Browser Support

- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile browsers (iOS Safari, Chrome Mobile)

## Technologies Used

- HTML5
- CSS3 (with modern features: backdrop-filter, clamp, animations)
- Google Fonts (Inter, Space Grotesk)

## License

© TeamLease EdTech | All Rights Reserved

## Support

For deployment assistance, refer to `DEPLOYMENT_GUIDE.md`

