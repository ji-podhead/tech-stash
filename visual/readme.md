# Matrix Screensaver using HTML and JS only - *the simplest way possible*

<div align="center">
  <h3><a href="https://github.com/ji-podhead/tech-stash/blob/main/visual/matrix.html">
    Full Code is on my git
  </a></h3>
</div>

![grafik](https://github.com/user-attachments/assets/2af1127d-f9c3-4504-a8f6-23bb258aac42)

When I created my CV in plain HTML(def. cool) i wanted to implement a little EasterEgg, so i thought of hiding some code to trick people and give it a little spice.
First i just wanted to hide a little decoded alert and just for fun i tried to recreate the matrix screen saver.

I asked AI to recreate it, after i created some pseudocode. 
- AI came up with some super weird algos and 3-4 linked list. To my suprise it actually worked xD, but it was doomed code from hell, so i wanted to do it better, but i kept the fancy canvas render technique (canvas api).

## Heres my ATTEMPT!

- first create a canvas in html.
  - we need a fixed window size, in order to spread the chars accross the screen, or we need to make this thing responsive which is not implemented in this approach.
```html
<div  style=" width:100%; height:100%;font-family: Georgia, serif;">
    <canvas id="matrix-container" style="z-index: 1; position: absolute;" width="768" height="1024"></canvas>

    <div style="z-index: 0; position: relative; width: 768px; height:1024px; justify-content: center; ">
    </div>
```

- we create a dict for the cols and define some static vars we need later (like font and screen size).
```js
        var canvas = document.getElementById('matrix-container');
        var ctx = canvas.getContext('2d');
        var width = 768;
        var height = 1024;
        var fontSize = 25;
        var columns = Math.floor(width / fontSize);
        var rows = Math.floor(height / fontSize);
        var matrix_dict = new Array(columns).fill(0).map(() => {
            offset=Math.floor(Math.random()*rows)
            return {
                y_offset_max:offset, //  tiefste mögliche y-position, bzw , max anzahl an buchstaben per row
                y_offset:0,  // Position des am höchsten Liegender Buchstabe in der Zeile
                y_speed:3+Math.floor(Math.random()*3), // die frequenz zur updaten der y-position in frames per second, zb F=5frames
                y_offset_iterator:0, // der iterator für die zeit bis zum updaten der y_position
                repspawn_speed: Math.floor(Math.random()*10), // die zeit bis nach dem kompletten auslöschen der spalte ein neues zeilenobjekt populiert wird
                respawn_iterator:0, // der iterator bis zum neuspawnen der spalte  nach vollständigem durchlauf
                fully_visible:false,  // könnte benutzt werden um zu checken ob der respawn_iterator inkrementiert werden soll
                letters:new Array(rows).fill(0).map(() => { 
                    return  {
                            char: String.fromCharCode(Math.floor(Math.random() * 128)), // der aktuelle Buchstabe der xy koordinate
                            time: 0, // der iterator zum updaten des chars
                            speed: 2+Math.floor(Math.random()*5), // die zeit bis zum updaten des chars in frames per second
                            color: '#'+Math.floor(Math.random()*16777215).toString(16),
                            alpha: 1
                        }  
                    })
                }
        })
```
### The render loop
- just like with particle systems, shaders or any other thing that gets animated, we need to have a loop to calculate the color, transformation, scale, rotation...
- the human eye sees 60 fps max, but 30 fps is fine ( any android game runs on 30 fps)
- based  on some rules we define, we move, respawn and colorize the chars in our render loop.

```js
 function calc() {
        var letters = new Array(columns).fill([]);
        var colors = new Array(columns).fill([]);
        var alphas = new Array(columns).fill([]);
        matrix_dict.forEach((col, columnIndex) => {
            col.y_offset_iterator++;
            if (col.y_offset_iterator >= col.y_speed) {
                col.y_offset++;
                col.y_offset_iterator = 0;
            }
            if (col.y_offset >= col.y_offset_max) {
                ofset=Math.floor(Math.random()*rows)
                col.y_offset_max= ofset
                col.y_offset = 0;
                col.fully_visible = false;
                col.respawn_iterator++;
                if (col.respawn_iterator >= col.repspawn_speed) {
                    col.letters = new Array(rows).fill(0).map(() => {
                    return {
                        char: String.fromCharCode(Math.floor(Math.random() * 128)),
                        time: 0,
                        speed: 2+Math.floor(Math.random() * 5)
                    }
                    });
                    col.respawn_iterator = 0;
                }
            }
            else{
                var row_array = new Array(col.y_offset).fill(' ');
                var color_array = new Array(col.y_offset).fill('fff');
                var alpha_array = new Array(col.y_offset).fill(0);
                col.letters.forEach((letter, rowIndex) => {
                    not_yet_visible=rowIndex >= col.y_offset  && col.fully_visible==false
                    if (rowIndex+col.y_offset >= col.y_offset_max || not_yet_visible) {
                        row_array.push("");
                        color_array.push("fff");
                        alpha_array.push(0);
                        if (rowIndex == col.y_offset && col.y_offset == col.y_offset_max && col.fully_visible == false) {
                            col.fully_visible = true;
                        }
                    }
                    else{
                        letter.time++;
                        if (letter.time >= letter.speed) {
                            letter.char = String.fromCharCode(Math.floor(Math.random() * 128));
                            letter.time = 0;
                            letter.speed=1+Math.floor(Math.random()*(50-col.y_speed))
                        }
                        row_array.push(letter.char);
                        color_array.push(letter.color);
                        alpha_array.push(letter.alpha);
                    }
            });
            letters[columnIndex] = row_array;
            colors[columnIndex] = color_array;
            alphas[columnIndex] = alpha_array;
           }
        });
        return {
        letters:letters,
        colors:colors,
        alphas:alphas}
        }
        function animate() {
            var results = calc();
            //console.log(results);
            render(results);
            setTimeout(animate, 100 / 60);
        }
        animate();
``` 
- notice that the char-replacement is random for any char because we have a random max-time value in our dict.
 - we can do this for any parameter, so we dont end up in super weird and complex linked list logic where we iterate a thousand times over the lists in a completely redundant way.
 - we only iterate twice over the matrix and that has a reason.
- i dont need the second loop, for the main logic, but think of what you can do with this:
  - LERP BETWEEN MULTIPLE MATRICES!!!
  - alter the matrix with another algo
  - bla bla
    
### Displaying the results

now that we have our color, char and alpha array, we pass it to our render function wich is just filling our canvas with the text.
 - you can do this line by line, or char by char
 - in my case i render it char by char because i wanted to have the possibilty to scale the chars individually, but you could just create a matrix for that, just like with did with the colors.
```js
        function render(matrix) {
            ctx.clearRect(0, 0, width, height);
            matrix.letters.forEach((col, columnIndex) => {
               // console.log(col)
                col.forEach((letter, rowIndex) => {
                    ctx.fillStyle = matrix.colors[columnIndex][rowIndex % matrix.colors[columnIndex].length]; 
                    ctx.fillText(letter, columnIndex * fontSize,  rowIndex * fontSize);
                    ctx.fillStyle = matrix.colors[columnIndex][rowIndex % matrix.colors[columnIndex].length];
                    ctx.shadowColor = 'rgba(1, 1, 0, 0.5)'; // Schattenfarbe
                    ctx.shadowBlur = 5; // Schatten-Blur-Effekt
                    ctx.shadowOffsetX = 2; // Schatten-Offset in x-Richtung
                    ctx.shadowOffsetY = 2; // Schatten-Offset in y-Richtung

                    //ctx.fillText(text, columnIndex * fontSize, 0);
                });
        });
        ctx.font = fontSize + 'px monospace';
        ctx.textAlign = 'left';
        }
```

### Summary

***Initialization***
 - The algorithm initializes a 2D array (matrix) with a fixed number of columns and rows. Each cell in the matrix represents a character on the screen.


***Character Generation:***
  - The algorithm generates a random character for each cell in the matrix. The characters are chosen from a set of printable ASCII characters.

***Color Generation***
 - The algorithm generates a random color for each character in the matrix. The colors are chosen from a set of possible colors.

***Render Loop***
- The algorithm enters a render loop, where it updates the characters, colors, and alpha values for each cell in the matrix.

***Character Update***
- The algorithm updates the character for each cell in the matrix based on a random speed value. The speed value determines how often the character is updated

***Rendering***
- The algorithm renders the updated characters, colors, and alpha values onto the screen using a canvas element.

### Thats it!
My algo is not perfect, im still rendering chars that are not even visible, so you could technically use the ofset value to alter the logic in order to avoid that.
However this algo is super lightweight, and with the fotn size of this example, it uses around 5-7% of my old shitty fx8300 cpu.

Have fun and happy hacking!
- ji 
