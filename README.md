<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Sorting Hat AI</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0; min-height: 100vh; padding: 20px 14px 40px;
    background: radial-gradient(circle at top, #2b1b4d, #0b0714 70%) fixed;
    font-family: Georgia, "Times New Roman", serif; color: #f1e6c8;
  }
  .card {
    width: 100%; max-width: 680px; margin: 0 auto; padding: 26px; border-radius: 16px;
    background: rgba(20, 12, 40, 0.9); border: 2px solid #c9a24b;
    box-shadow: 0 0 30px rgba(201, 162, 75, 0.35);
  }
  h1 { text-align: center; margin: 0 0 4px; color: #e8c766; letter-spacing: 2px; font-size: 26px; }
  .sub { text-align: center; font-style: italic; opacity: .85; }
  .tag { text-align: center; font-size: 14px; opacity: .7; margin: 6px 0 18px; }
  h2 { font-size: 17px; margin: 22px 0 8px; color: #e8c766; }
  input[type=text] {
    width: 100%; padding: 10px; border-radius: 8px; border: 1px solid #c9a24b;
    background: #1a1030; color: #f1e6c8; font-size: 16px; font-family: inherit;
  }
  .ages { display: grid; grid-template-columns: repeat(auto-fit, minmax(135px, 1fr)); gap: 8px; }
  .ages label {
    display: block; padding: 10px; text-align: center; border: 1px solid #c9a24b; border-radius: 8px;
    cursor: pointer; font-size: 14px; background: #1a1030;
  }
  .ages input { position: absolute; opacity: 0; pointer-events: none; }
  .ages input:checked + span { color: #1a1030; }
  .ages label:has(input:checked) { background: #c9a24b; color: #1a1030; font-weight: bold; }
  .ages label:has(input:focus-visible) { outline: 2px solid #ffe08a; outline-offset: 2px; }
  .sliders { display: grid; gap: 10px 22px; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr)); }
  .row { display: flex; flex-direction: column; gap: 3px; }
  .row .top { display: flex; justify-content: space-between; font-size: 15px; }
  .row .val { color: #e8c766; font-weight: bold; min-width: 1.5em; text-align: right; }
  input[type=range] { width: 100%; accent-color: #c9a24b; }
  .scale { display: flex; justify-content: space-between; font-size: 11px; opacity: .55; }
  button.go {
    width: 100%; margin-top: 22px; padding: 13px; font-size: 18px; cursor: pointer;
    border: none; border-radius: 8px; background: #c9a24b; color: #1a1030; font-family: inherit; font-weight: bold;
  }
  button.go:hover { background: #e8c766; }
  #msg { margin-top: 14px; padding: 10px; border-radius: 8px; background: #3a2a10; display: none; text-align: center; }
  #results { margin-top: 20px; display: none; }
  .res { padding: 16px; border-radius: 12px; margin-bottom: 12px; border: 1px solid rgba(201,162,75,.5); background: rgba(255,255,255,.04); animation: pop .5s ease; }
  .res.first { background: rgba(201,162,75,.14); border: 2px solid #c9a24b; }
  .res .head { display: flex; justify-content: space-between; align-items: baseline; gap: 10px; flex-wrap: wrap; }
  .res .name { font-size: 20px; font-weight: bold; color: #ffe08a; }
  .res .pct { font-size: 15px; opacity: .9; }
  .bar { height: 8px; border-radius: 4px; background: #1a1030; margin: 8px 0 10px; overflow: hidden; }
  .bar i { display: block; height: 100%; background: linear-gradient(90deg, #a8791f, #ffe08a); border-radius: 4px; }
  .res p { margin: 6px 0; line-height: 1.5; font-size: 15px; }
  .why { font-size: 13px; opacity: .8; font-style: italic; }
  details { margin-top: 20px; padding: 12px 14px; border: 1px dashed #c9a24b; border-radius: 10px; font-size: 14px; line-height: 1.55; }
  summary { cursor: pointer; color: #e8c766; font-weight: bold; }
  .note { margin-top: 16px; font-size: 12px; text-align: center; opacity: .6; line-height: 1.5; }
  @keyframes pop { from { transform: scale(.94); opacity: 0; } to { transform: scale(1); opacity: 1; } }
</style>
</head>
<body>
<div class="card">
  <h1>SORTING HAT AI</h1>
  <div class="sub">Where Code Meets Magic</div>
  <div class="tag">A machine-learning guide for any age: tell the hat what you love, and it suggests where to go next.</div>

  <label for="name" style="font-weight:bold">Your name (optional)</label>
  <input type="text" id="name" placeholder="Enter your name">

  <h2>1. Who is the hat guiding?</h2>
  <div class="ages" id="ages"></div>

  <h2>2. How much do you enjoy each of these? (0 = not at all, 5 = love it)</h2>
  <div class="sliders" id="sliders"></div>

  <button class="go" onclick="ask()">Ask the Hat</button>
  <div id="msg"></div>
  <div id="results"></div>

  <details>
    <summary>How does this work? (the machine learning part)</summary>
    <p id="modelinfo">Training the model...</p>
    <p>The hat is a <b>multinomial logistic regression</b> classifier (softmax regression). It learns one weight for each pair of interest and pathway, and turns your 10 slider values into a probability for each of 9 pathways. Training runs right here in your browser, using gradient descent, every time the page loads. No data leaves your device.</p>
    <p>Your age group does not change the prediction. It only decides which kind of suggestion is shown: activities for children, study and club ideas for teens, career moves for adults, and hobbies, mentoring and volunteering for seniors.</p>
    <p><b>Honest limits:</b> the training examples are generated from hand-defined interest profiles with random variation. They are not real people's records, so treat the output as a starting point for thinking, not a verdict. Talk to a counsellor, teacher or mentor too.</p>
  </details>
  <div class="note">Built by Spell Squad for Triwizathon 2026.</div>
</div>

<script>
// ==ML-START==
const FEATURES = [
  "Technology & computers", "Art & design", "Science & experiments", "Helping people",
  "Business & money", "Nature & animals", "Sports & fitness", "Music & performing",
  "Writing & stories", "Building & fixing things"
];

// Hand-defined "typical interest profile" for each pathway, on a 0-5 scale (same order as FEATURES).
const CLASSES = [
  { id: "tech",     name: "Technology & AI",             centroid: [5,2,3,1,2,0,0,1,1,3] },
  { id: "creative", name: "Creative Arts & Design",      centroid: [2,5,1,1,1,1,0,3,3,2] },
  { id: "science",  name: "Science & Research",          centroid: [3,1,5,2,0,3,0,0,2,2] },
  { id: "care",     name: "Care, Health & Teaching",     centroid: [0,1,2,5,1,2,1,1,3,0] },
  { id: "business", name: "Business & Entrepreneurship", centroid: [3,1,0,2,5,0,1,1,2,0] },
  { id: "nature",   name: "Nature & Environment",        centroid: [0,2,3,2,0,5,3,0,1,2] },
  { id: "sport",    name: "Sports & Fitness",            centroid: [0,0,1,3,1,2,5,1,0,1] },
  { id: "media",    name: "Media, Music & Writing",      centroid: [1,3,0,2,2,0,0,5,5,0] },
  { id: "build",    name: "Engineering & Hands-on Skills", centroid: [3,2,2,0,1,1,1,0,0,5] }
];

const AGE_GROUPS = [
  { id: "kid",    label: "Child (6-12)" },
  { id: "teen",   label: "Teen (13-17)" },
  { id: "adult",  label: "Adult (18-59)" },
  { id: "senior", label: "Senior (60+)" }
];

const SUGGESTIONS = {
  tech: {
    kid: "Try Scratch or Code.org games, build a simple robot kit, join a coding club.",
    teen: "Learn Python or web basics, join a hackathon, explore Computer Science or IT streams.",
    adult: "Software developer, data analyst or AI/ML engineer. An online course or bootcamp is a fast way to switch.",
    senior: "Learn to build a simple website or use AI tools, join a digital-literacy volunteer group, mentor others in tech."
  },
  creative: {
    kid: "Drawing, craft, stop-motion films with a phone, join an art club.",
    teen: "Build a design portfolio, learn Canva or Figma, explore design, animation or fine arts courses.",
    adult: "UI/UX designer, animator, content creator or architect. Freelance projects are a good way to start.",
    senior: "Painting or pottery classes, digital photography, selling crafts online."
  },
  science: {
    kid: "Kitchen-science experiments, science museum visits, a microscope or telescope.",
    teen: "Science olympiads, lab-based projects, explore Physics, Chemistry, Biology or research streams.",
    adult: "Researcher, lab scientist, data scientist or lab technician. A postgraduate degree or research-assistant role can open the door.",
    senior: "Citizen-science projects, astronomy clubs, science communication, volunteering at a science centre."
  },
  care: {
    kid: "Class-helper or buddy roles, first-aid basics, reading to younger children.",
    teen: "Volunteer at an NGO, explore Biology, Psychology or Education streams, try peer tutoring.",
    adult: "Teacher, nurse, counsellor, social worker or doctor. Look at certificate and degree routes.",
    senior: "Mentoring, tutoring, hospital or NGO volunteering, community support groups."
  },
  business: {
    kid: "Run a small craft or lemonade stall, learn to save with a piggy bank.",
    teen: "Start a mini-venture, join an entrepreneurship club, explore Commerce and Economics.",
    adult: "Entrepreneur, marketing, finance or product manager. Test a side business before you commit.",
    senior: "Consult in your own field, start a small home business, advise young founders."
  },
  nature: {
    kid: "Gardening, bird-watching, pet care, nature walks with a journal.",
    teen: "Join an eco club, explore Environmental Science, Agriculture or Veterinary routes.",
    adult: "Environmental scientist, agri-entrepreneur, forestry, wildlife or veterinary roles.",
    senior: "Gardening, organic farming, conservation volunteering, bird counts."
  },
  sport: {
    kid: "Try several sports, join a local team, swimming or athletics.",
    teen: "Compete at school level, explore sports science and physical-education courses.",
    adult: "Coach, fitness trainer, physiotherapist, sports analyst or sports manager.",
    senior: "Yoga, walking groups, masters sports leagues, volunteer coaching."
  },
  media: {
    kid: "Storytelling, school plays, music lessons, a diary or comic of your own.",
    teen: "Start a blog or YouTube channel, join drama or music clubs, explore Mass Communication or Journalism.",
    adult: "Content creator, writer, journalist, musician, video editor or voice artist.",
    senior: "Write a memoir, join a choir or theatre group, start a blog or podcast."
  },
  build: {
    kid: "LEGO and building kits, simple electronics, making things from recycled materials.",
    teen: "Robotics club, workshop projects, explore Engineering or ITI/Diploma routes.",
    adult: "Engineer, electrician, technician or product maker. Skill certificates are a fast route in.",
    senior: "Repair cafes, woodworking, teaching practical skills to young people."
  }
};

// ---- small seeded random generator so training is the same on every device ----
function mulberry32(a) {
  return function () {
    a |= 0; a = (a + 0x6D2B79F5) | 0;
    let t = Math.imul(a ^ (a >>> 15), 1 | a);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}
function gauss(rand) {
  let u = 0, v = 0;
  while (u === 0) u = rand();
  while (v === 0) v = rand();
  return Math.sqrt(-2 * Math.log(u)) * Math.cos(2 * Math.PI * v);
}

// ---- dataset: samples scattered around each pathway's typical profile ----
function buildDataset(perClass, rand) {
  const X = [], Y = [];
  CLASSES.forEach((c, k) => {
    for (let i = 0; i < perClass; i++) {
      const x = c.centroid.map(v => {
        let n = v + gauss(rand) * 1.0;
        if (rand() < 0.12) n += rand() * 4 - 2;      // occasional unusual interest
        return Math.min(5, Math.max(0, Math.round(n)));
      });
      X.push(x.map(v => v / 5));
      Y.push(k);
    }
  });
  return { X, Y };
}

function shuffleSplit(X, Y, testFrac, rand) {
  const idx = X.map((_, i) => i);
  for (let i = idx.length - 1; i > 0; i--) {
    const j = Math.floor(rand() * (i + 1));
    [idx[i], idx[j]] = [idx[j], idx[i]];
  }
  const nTest = Math.round(idx.length * testFrac);
  const pick = ids => ({ X: ids.map(i => X[i]), Y: ids.map(i => Y[i]) });
  return { test: pick(idx.slice(0, nTest)), train: pick(idx.slice(nTest)) };
}

function softmax(z) {
  const m = Math.max(...z);
  const e = z.map(v => Math.exp(v - m));
  const s = e.reduce((a, b) => a + b, 0);
  return e.map(v => v / s);
}

function scores(model, x) {
  return model.W.map((row, k) => row.reduce((a, w, j) => a + w * x[j], model.b[k]));
}
function predictProba(model, x) { return softmax(scores(model, x)); }

// ---- training: multinomial logistic regression by batch gradient descent ----
function train(X, Y, K, epochs, lr, l2) {
  const D = X[0].length, n = X.length;
  const W = Array.from({ length: K }, () => new Array(D).fill(0));
  const b = new Array(K).fill(0);
  for (let e = 0; e < epochs; e++) {
    const gW = Array.from({ length: K }, () => new Array(D).fill(0));
    const gb = new Array(K).fill(0);
    for (let i = 0; i < n; i++) {
      const p = softmax(W.map((row, k) => row.reduce((a, w, j) => a + w * X[i][j], b[k])));
      for (let k = 0; k < K; k++) {
        const g = p[k] - (Y[i] === k ? 1 : 0);
        gb[k] += g;
        for (let j = 0; j < D; j++) gW[k][j] += g * X[i][j];
      }
    }
    for (let k = 0; k < K; k++) {
      b[k] -= lr * gb[k] / n;
      for (let j = 0; j < D; j++) W[k][j] -= lr * (gW[k][j] / n + l2 * W[k][j]);
    }
  }
  return { W, b };
}

function accuracy(model, X, Y) {
  let ok = 0;
  X.forEach((x, i) => {
    const p = predictProba(model, x);
    if (p.indexOf(Math.max(...p)) === Y[i]) ok++;
  });
  return ok / X.length;
}

function trainHat() {
  const rand = mulberry32(2026);
  const { X, Y } = buildDataset(150, rand);
  const { train: tr, test: te } = shuffleSplit(X, Y, 0.2, rand);
  const model = train(tr.X, tr.Y, CLASSES.length, 150, 4.0, 0.001);
  model.stats = {
    trainN: tr.X.length, testN: te.X.length,
    trainAcc: accuracy(model, tr.X, tr.Y), testAcc: accuracy(model, te.X, te.Y)
  };
  return model;
}
// ==ML-END==

// ---------------- page code ----------------
let MODEL = null;

function buildUI() {
  const ages = document.getElementById("ages");
  AGE_GROUPS.forEach((a, i) => {
    const l = document.createElement("label");
    l.innerHTML = '<input type="radio" name="age" value="' + a.id + '"' + (i === 2 ? " checked" : "") + '><span>' + a.label + '</span>';
    ages.appendChild(l);
  });

  const box = document.getElementById("sliders");
  FEATURES.forEach((f, i) => {
    const d = document.createElement("div");
    d.className = "row";
    d.innerHTML =
      '<div class="top"><label for="s' + i + '">' + f + '</label><span class="val" id="v' + i + '">0</span></div>' +
      '<input type="range" id="s' + i + '" min="0" max="5" step="1" value="0">' +
      '<div class="scale"><span>0</span><span>5</span></div>';
    box.appendChild(d);
    d.querySelector("input").addEventListener("input", e => {
      document.getElementById("v" + i).textContent = e.target.value;
    });
  });
}

function ask() {
  const msg = document.getElementById("msg");
  const out = document.getElementById("results");
  const vals = FEATURES.map((_, i) => Number(document.getElementById("s" + i).value));
  const total = vals.reduce((a, b) => a + b, 0);
  if (total < 3 || vals.filter(v => v > 0).length < 2) {
    msg.style.display = "block";
    msg.textContent = "Move at least two sliders above 0 so the hat has something to work with.";
    out.style.display = "none";
    return;
  }
  msg.style.display = "none";

  const name = document.getElementById("name").value.trim();
  const age = document.querySelector('input[name="age"]:checked').value;
  const x = vals.map(v => v / 5);
  const p = predictProba(MODEL, x);
  const order = p.map((v, i) => [v, i]).sort((a, b) => b[0] - a[0]).slice(0, 3).filter((t, r) => r === 0 || t[0] >= 0.05);

  let html = "<h2>" + (name ? name + ", the hat suggests..." : "The hat suggests...") + "</h2>";
  if (order[0][0] < 0.4) {
    html += '<p class="why">Your interests are spread across many areas, so no single pathway stands out. These are the closest matches; it is fine to explore more than one.</p>';
  }
  order.forEach(([prob, k], rank) => {
    const c = CLASSES[k];
    const contrib = FEATURES.map((f, j) => [MODEL.W[k][j] * x[j], f, vals[j]])
      .filter(t => t[2] > 0).sort((a, b) => b[0] - a[0]).slice(0, 2).map(t => t[1].toLowerCase());
    html +=
      '<div class="res' + (rank === 0 ? " first" : "") + '">' +
        '<div class="head"><span class="name">' + (rank + 1) + ". " + c.name + '</span><span class="pct">' + Math.round(prob * 100) + '% match</span></div>' +
        '<div class="bar"><i style="width:' + Math.max(3, Math.round(prob * 100)) + '%"></i></div>' +
        '<p>' + SUGGESTIONS[c.id][age] + '</p>' +
        (contrib.length ? '<p class="why">Why: mainly your interest in ' + contrib.join(" and ") + '.</p>' : "") +
      '</div>';
  });
  out.innerHTML = html;
  out.style.display = "block";
  out.scrollIntoView({ behavior: "smooth", block: "start" });
}

window.addEventListener("DOMContentLoaded", () => {
  buildUI();
  MODEL = trainHat();
  const s = MODEL.stats;
  document.getElementById("modelinfo").innerHTML =
    "Trained on <b>" + s.trainN + "</b> examples, tested on <b>" + s.testN + "</b> unseen examples. " +
    "Accuracy on the unseen test set: <b>" + (s.testAcc * 100).toFixed(1) + "%</b> " +
    "(training: " + (s.trainAcc * 100).toFixed(1) + "%).";
});
</script>
</body>
</html>
