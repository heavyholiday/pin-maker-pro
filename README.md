# pin-maker-pro
git init
git add .
git commit -m "first version"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/pin-maker-pro.git
git push -u origin main
import { useEffect, useRef, useState } from "react";
import { fabric } from "fabric";
import jsPDF from "jspdf";

const BUTTON_SIZES = {
  "1": { cut: 1.75, safe: 0.75, spacing: 1.9 },
  "1.5": { cut: 2.0, safe: 1.25, spacing: 2.2 },
  "2.25": { cut: 2.75, safe: 2.0, spacing: 2.8 },
  "3": { cut: 3.5, safe: 2.75, spacing: 3.6 }
};

export default function App() {
  const canvasRef = useRef(null);
  const [canvas, setCanvas] = useState(null);
  const [size, setSize] = useState("2.25");
  const [designs, setDesigns] = useState([]);

  useEffect(() => {
    const c = new fabric.Canvas("canvas", {
      width: 500,
      height: 500,
      backgroundColor: "#fff"
    });

    canvasRef.current = c;
    setCanvas(c);

    drawGuides(c, size);
  }, []);

  const drawGuides = (c, sizeKey) => {
    const data = BUTTON_SIZES[sizeKey];
    const scale = 100;

    c.clear();

    const cut = new fabric.Circle({
      radius: (data.cut / 2) * scale,
      stroke: "black",
      fill: "",
      selectable: false
    });

    const safe = new fabric.Circle({
      radius: (data.safe / 2) * scale,
      stroke: "green",
      fill: "",
      selectable: false
    });

    [cut, safe].forEach(circle => {
      circle.set({
        left: 250,
        top: 250,
        originX: "center",
        originY: "center"
      });
      c.add(circle);
    });
  };

  const changeSize = (newSize) => {
    setSize(newSize);
    drawGuides(canvasRef.current, newSize);
  };

  const uploadImage = (e) => {
    const file = e.target.files[0];
    const reader = new FileReader();

    reader.onload = (f) => {
      fabric.Image.fromURL(f.target.result, (img) => {
        img.scaleToWidth(200);
        img.set({ left: 150, top: 150 });
        canvasRef.current.add(img);
      });
    };

    reader.readAsDataURL(file);
  };

  const addText = () => {
    const text = new fabric.Textbox("Your Text", {
      left: 150,
      top: 150,
      fontSize: 24
    });
    canvasRef.current.add(text);
  };

  const addToSheet = () => {
    const img = canvasRef.current.toDataURL("image/png");
    setDesigns([...designs, img]);
  };

  const exportPDF = () => {
    const data = BUTTON_SIZES[size];
    const pdf = new jsPDF({ unit: "in", format: "letter" });

    const images = designs.length > 0 ? designs : [canvasRef.current.toDataURL("image/png")];

    let x = 0.5;
    let y = 0.5;

    images.forEach((img, i) => {
      pdf.addImage(img, "PNG", x, y, data.cut, data.cut);

      x += data.spacing;

      if (x + data.cut > 8.5) {
        x = 0.5;
        y += data.spacing;
      }
    });

    pdf.save("pin-maker-pro.pdf");
  };

  return (
    <div style={{ textAlign: "center", padding: "20px" }}>
      <h1>Pin Maker Pro 🔘</h1>
      <p>Create perfect button templates in seconds</p>

      <select onChange={(e) => changeSize(e.target.value)} value={size}>
        <option value="1">1 Inch</option>
        <option value="1.5">1.5 Inch</option>
        <option value="2.25">2.25 Inch</option>
        <option value="3">3 Inch</option>
      </select>

      <br /><br />

      <input type="file" onChange={uploadImage} />
      <button onClick={addText}>Add Text</button>
      <button onClick={addToSheet}>Add To Sheet</button>
      <button onClick={exportPDF}>Download PDF</button>

      <br /><br />

      <canvas id="canvas" />
    </div>
  );
}
