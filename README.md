<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Sai Chandana — Interactive Wheel Portfolio</title>
<style>
  :root{
    --bg:#0f1724;
    --card:#0b1020;
    --accent:#6ee7b7;
    --muted:#94a3b8;
    --glass: rgba(255,255,255,0.04);
    --glass-2: rgba(255,255,255,0.02);
    --glass-border: rgba(255,255,255,0.04);
    --shadow: 0 8px 24px rgba(2,6,23,0.6);
  }
  html,body{height:100%;margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial; background:linear-gradient(180deg,#071022 0%, #0f1724 100%); color:#e6eef8}
  .wrap{
    min-height:100vh;
    display:flex;
    align-items:center;
    justify-content:center;
    gap:2rem;
    padding:3rem;
    box-sizing:border-box;
  }

  /* left column: wheel card */
  .card{
    width:min(540px,92vw);
    background:linear-gradient(180deg,var(--card), rgba(6,10,18,0.6));
    border-radius:18px;
    padding:24px;
    box-shadow:var(--shadow);
    border:1px solid var(--glass-border);
    display:flex;
    gap:18px;
    align-items:center;
    justify-content:center;
    flex-direction:column;
  }
  header h1{margin:0;font-size:1.25rem}
  header p{margin:6px 0 0;color:var(--muted);font-size:0.92rem}

  .wheel-stage{
    width:360px;
    height:360px;
    display:grid;
    place-items:center;
    position:relative;
  }

  /* rotating container */
  .wheel{
    width:320px;height:320px;
    transform-origin:center;
    transition:transform 600ms cubic-bezier(.2,.9,.2,1);
    will-change:transform;
    border-radius:50%;
    display:block;
    position:relative;
    filter:drop-shadow(0 6px 18px rgba(3,7,18,0.6));
  }

  /* continuous rotation */
  .spin {
    animation:spin 18s linear infinite;
    animation-play-state:running;
  }
  .paused.spin{ animation-play-state:paused; }

  @keyframes spin {
    from{ transform: rotate(0deg) }
    to{ transform: rotate(360deg) }
  }

  /* center knob */
  .center-knob{
    position:absolute;
    width:86px;height:86px;border-radius:50%;
    left:50%;top:50%;transform:translate(-50%,-50%);
    display:flex;align-items:center;justify-content:center;
    background:linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.01));
    border:1px solid var(--glass-border);
    box-shadow:0 6px 18px rgba(2,6,23,0.45);
    cursor:grab;
    user-select:none;
  }
  .center-knob:active{cursor:grabbing}
  .center-knob h2{margin:0;font-size:0.9rem; color:var(--accent)}

  /* legend / instructions */
  .legend{
    width:320px;
    color:var(--muted);
    font-size:0.92rem;
    text-align:center;
    margin-top:8px;
  }

  /* modal */
  .modal-backdrop{
    position:fixed;inset:0;display:none;align-items:center;justify-content:center;
    background:linear-gradient(rgba(2,6,23,0.6), rgba(2,6,23,0.6));
    z-index:60;padding:24px;
  }
  .modal-backdrop.show{ display:flex; }
  .modal{
    max-width:720px;width:100%;background:linear-gradient(180deg,#071225,#06101b);
    border-radius:12px;padding:20px;border:1px solid var(--glass-border);box-shadow:0 14px 40px rgba(2,6,23,0.7);
  }
  .modal h3{margin:0 0 8px}
  .modal p{margin:0 0 12px;color:var(--muted)}
  .close-btn{
    background:transparent;border:1px solid var(--glass-border);color:var(--muted);
    padding:8px 12px;border-radius:8px;cursor:pointer;float:right;
  }

  /* slice pointer highlight indicator text box (small) */
  .info-chip{
    position:absolute;left:50%;transform:translateX(-50%);bottom:-28px;background:var(--glass);border:1px solid var(--glass-border);
    padding:6px 12px;border-radius:999px;color:var(--accent);font-weight:600;font-size:0.86rem;
    box-shadow:0 6px 18px rgba(2,6,23,0.5);
  }

  /* responsive */
  @media (max-width:720px){
    .wrap{padding:1.4rem;gap:1rem;flex-direction:column-reverse}
    .legend{width:92%}
  }

  /* small visual helpers for slice outlines when active */
  .slice-active { filter: drop-shadow(0 12px 30px rgba(110,231,183,0.08)); }
  .slice-dim { opacity:0.45; transition:opacity .25s linear; }

  /* clickable pointer cursor on slices */
  svg .slice { cursor:pointer; transition:transform .18s ease, opacity .18s ease; }
  svg .slice:hover { transform: scale(1.02); }
</style>
</head>
<body>
  <div class="wrap" role="main">
    <div class="card" aria-label="Interactive wheel card">
      <header style="width:100%;display:flex;flex-direction:column;align-items:center">
        <h1>Hi, I'm <strong>Sai Chandana</strong> 👋</h1>
        <p>Full-Stack & AI Engineer — click the wheel slices to explore projects & contact</p>
      </header>

      <div class="wheel-stage" id="stage">
        <!-- SVG wheel (6 slices) -->
        <svg id="wheel" class="wheel spin" viewBox="0 0 400 400" role="img" aria-label="Interactive portfolio wheel" xmlns="http://www.w3.org/2000/svg">
          <!-- We create 6 slices using paths and group them with data attributes -->
          <!-- colors chosen to be elegant + subtle -->
          <defs>
            <filter id="soft" x="-50%" y="-50%" width="200%" height="200%">
              <feGaussianBlur stdDeviation="2" />
            </filter>
          </defs>

          <!-- circle background -->
          <circle cx="200" cy="200" r="196" fill="url(#bggrad)" stroke="rgba(255,255,255,0.03)" stroke-width="1" />
          <!-- slices -->
          <!-- Each slice uses an arc path. We attach data-title & data-content to each group. -->
          <g id="slice-1" class="slice" data-title="Work Nexus" data-content="Modern workspace app built with Next.js, Clerk and real-time tools. Live demos & architecture in repository.">
            <path d="M200 200 L200 4 A196 196 0 0 1 358.5 78.5 z" fill="#0ea5a3" />
          </g>

          <g id="slice-2" class="slice" data-title="Robotics" data-content="Autonomous robotic systems: ROS2 + Gazebo simulations for obstacle avoidance, SLAM, and navigation.">
            <path d="M200 200 L358.5 78.5 A196 196 0 0 1 358.5 321.5 z" fill="#7dd3fc" />
          </g>

          <g id="slice-3" class="slice" data-title="ML Models" data-content="Classification models (KNN, Random Forest, Logistic) with high accuracy and cross-validation.">
            <path d="M200 200 L358.5 321.5 A196 196 0 0 1 200 396 z" fill="#a78bfa" />
          </g>

          <g id="slice-4" class="slice" data-title="Secure Systems" data-content="Full-stack AES-256 encryption system: Node.js backend, PostgreSQL, secure auth flows.">
            <path d="M200 200 L200 396 A196 196 0 0 1 41.5 321.5 z" fill="#f0abfc" />
          </g>

          <g id="slice-5" class="slice" data-title="Frontend & Web" data-content="Next.js / React projects, performant UI, Tailwind-ready components, responsive design.">
            <path d="M200 200 L41.5 321.5 A196 196 0 0 1 41.5 78.5 z" fill="#f97316" />
          </g>

          <g id="slice-6" class="slice" data-title="Contact / CV" data-content="Portfolio, resume and contact details. Open to FAANG and research roles.">
            <path d="M200 200 L41.5 78.5 A196 196 0 0 1 200 4 z" fill="#60a5fa" />
          </g>

          <!-- subtle center circle -->
          <circle cx="200" cy="200" r="86" fill="rgba(255,255,255,0.02)" stroke="rgba(255,255,255,0.04)" stroke-width="1"/>

        </svg>

        <div class="center-knob" id="knob" title="Drag or click slices">
          <h2>Explore</h2>
        </div>

        <div class="info-chip" id="chip" style="display:none" aria-hidden="true">Click a slice</div>
      </div>

      <div class="legend">
        Tip: The wheel rotates slowly. Click any slice to pause & open details. Press <kbd>Esc</kbd> to close the panel.
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div class="modal-backdrop" id="backdrop" role="dialog" aria-modal="true" aria-hidden="true">
    <div class="modal" role="document" aria-labelledby="modal-title">
      <button class="close-btn" id="closeBtn" aria-label="Close details">Close</button>
      <h3 id="modal-title"></h3>
      <p id="modal-desc"></p>
      <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:12px">
        <a id="link-repo" href="#" target="_blank" rel="noopener" style="text-decoration:none;padding:8px 12px;border-radius:10px;border:1px solid var(--glass-border);color:var(--accent)">View Repo</a>
        <a href="https://saichandanajampala.netlify.app/" target="_blank" rel="noopener" style="text-decoration:none;padding:8px 12px;border-radius:10px;border:1px solid var(--glass-border);color:var(--muted)">Portfolio</a>
        <a href="mailto:saichandanajampala@gmail.com" style="text-decoration:none;padding:8px 12px;border-radius:10px;border:1px solid var(--glass-border);color:var(--muted)">Email</a>
      </div>
    </div>
  </div>

<script>
  // JS to make the wheel interactive
  (function(){
    const wheel = document.getElementById('wheel');
    const slices = Array.from(document.querySelectorAll('.slice'));
    const backdrop = document.getElementById('backdrop');
    const modalTitle = document.getElementById('modal-title');
    const modalDesc = document.getElementById('modal-desc');
    const chip = document.getElementById('chip');
    const closeBtn = document.getElementById('closeBtn');
    const linkRepo = document.getElementById('link-repo');

    // map slice ids to repo links (update as desired)
    const repoMap = {
      'slice-1': 'https://github.com/sai-chandana-jampala/Work-Nexus',
      'slice-2': 'https://github.com/sai-chandana-jampala/ALL_CLG_PROJECTS',
      'slice-3': 'https://github.com/sai-chandana-jampala/ALL_MACHINE_LEARNING_PROJECTS',
      'slice-4': 'https://github.com/sai-chandana-jampala/Backend---applications',
      'slice-5': 'https://github.com/sai-chandana-jampala',
      'slice-6': 'https://saichandanajampala.netlify.app/'
    };

    // show a small chip hint briefly
    setTimeout(()=>{ chip.style.display='block'; chip.style.opacity=1; chip.style.transition='opacity .6s'; }, 600);
    setTimeout(()=>{ chip.style.opacity=0; setTimeout(()=>chip.style.display='none',400); }, 4200);

    let activeId = null;

    // clicking a slice
    slices.forEach(s => {
      s.addEventListener('click', e => {
        const id = s.id;
        const title = s.dataset.title || 'Details';
        const content = s.dataset.content || '';
        // pause the wheel
        wheel.classList.add('paused');
        // highlight active slice & dim others
        slices.forEach(x => { if (x.id === id) x.classList.add('slice-active'); else x.classList.add('slice-dim'); });
        activeId = id;
        openModal(title, content, repoMap[id] || '#');
      });
    });

    function openModal(title, content, repoUrl){
      modalTitle.textContent = title;
      modalDesc.textContent = content;
      linkRepo.href = repoUrl;
      backdrop.classList.add('show');
      backdrop.setAttribute('aria-hidden','false');
      // focus management
      closeBtn.focus();
    }

    function closeModal(){
      backdrop.classList.remove('show');
      backdrop.setAttribute('aria-hidden','true');
      // resume spinning
      wheel.classList.remove('paused');
      // clear highlights
      slices.forEach(x => { x.classList.remove('slice-active','slice-dim'); });
      activeId = null;
    }

    closeBtn.addEventListener('click', closeModal);
    backdrop.addEventListener('click', (ev)=>{ if(ev.target === backdrop) closeModal(); });
    document.addEventListener('keydown', (ev) => {
      if(ev.key === 'Escape') closeModal();
    });

    // optional: drag-to-rotate knob (basic)
    const knob = document.getElementById('knob');
    let dragging = false, lastX=0, currentRot = 0;
    knob.addEventListener('pointerdown', e => {
      dragging = true; lastX = e.clientX;
      wheel.classList.add('paused'); // pause while dragging
      knob.setPointerCapture(e.pointerId);
    });
    window.addEventListener('pointermove', e => {
      if(!dragging) return;
      const dx = e.clientX - lastX;
      lastX = e.clientX;
      currentRot += dx * 0.35;
      wheel.style.transform = `rotate(${currentRot}deg)`;
    });
    window.addEventListener('pointerup', e => {
      if(dragging){
        dragging=false;
        // allow wheel to resume spinning but keep rotation transform applied smoothly
        // compute transform matrix and then remove explicit transform to let animation continue from current rotation:
        // For simplicity, keep current rotation and continue animation by clearing inline style after short delay
        setTimeout(()=>{ wheel.style.transform = ''; wheel.classList.remove('paused') }, 120);
        knob.releasePointerCapture && knob.releasePointerCapture(e.pointerId);
      }
    });

    // ensure tab focusable and clickable via keyboard
    slices.forEach(s => {
      s.setAttribute('tabindex','0');
      s.addEventListener('keydown', (e)=>{
        if(e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          s.click();
        }
      });
    });

    // small accessibility: announce when modal opens (screen readers)
    backdrop.addEventListener('transitionstart', ()=> {
      if(backdrop.classList.contains('show')) backdrop.setAttribute('aria-hidden','false');
    });

    // Prevent scrolling when modal open
    const observer = new MutationObserver(() => {
      if(backdrop.classList.contains('show')) document.body.style.overflow = 'hidden';
      else document.body.style.overflow = '';
    });
    observer.observe(backdrop, { attributes:true, attributeFilter:['class'] });

  })();
</script>
</body>
</html>
