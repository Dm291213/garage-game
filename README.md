<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Гаражный нумеролог</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: 'Segoe UI', Tahoma, sans-serif;
    background: radial-gradient(circle at 50% 0%, #1b2735 0%, #090a0f 100%);
    color: #e8e8e8;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    padding: 20px;
  }
  .game { width: 100%; max-width: 820px; }

  h1 {
    text-align: center; font-size: 26px; margin-bottom: 4px;
    background: linear-gradient(90deg, #f5d76e, #f8b500, #f5d76e);
    -webkit-background-clip: text; -webkit-text-fill-color: transparent;
    cursor: default; user-select: none;
  }
  .subtitle { text-align: center; color: #8a94a6; font-size: 13px; margin-bottom: 18px; }

  .events-banner { display: flex; gap: 8px; margin-bottom: 14px; flex-wrap: wrap; justify-content: center; }
  .event-chip {
    display: flex; align-items: center; gap: 6px;
    padding: 6px 12px; border-radius: 20px;
    font-size: 12px; font-weight: 700;
  }
  .event-chip.candy  { background: linear-gradient(135deg,#ff6bcb,#ff8fdb); color:#3a0024; box-shadow: 0 0 16px rgba(255,107,203,0.6); }
  .event-chip.lucky  { background: linear-gradient(135deg,#6ee7a8,#34d399); color:#052e1b; box-shadow: 0 0 16px rgba(110,231,168,0.6); }
  .event-chip.money  { background: linear-gradient(135deg,#f5d76e,#f8b500); color:#1b2735; box-shadow: 0 0 16px rgba(245,215,110,0.6); }
  .event-chip.mega   { background: linear-gradient(135deg,#a855f7,#ec4899); color:#fff;    box-shadow: 0 0 16px rgba(168,85,247,0.7); }
  .event-chip .timer { font-weight: 400; opacity: 0.75; }

  .welcome-overlay {
    position: fixed; inset: 0;
    background: rgba(9, 10, 15, 0.85);
    backdrop-filter: blur(6px);
    display: flex; align-items: center; justify-content: center;
    z-index: 1000;
  }
  .welcome-overlay.hide { display: none; }
  .welcome-card {
    background: linear-gradient(160deg, rgba(30,40,55,0.95), rgba(15,20,30,0.98));
    border: 1px solid rgba(245,215,110,0.4);
    border-radius: 20px; padding: 36px 40px;
    max-width: 460px; text-align: center;
    box-shadow: 0 20px 60px rgba(0,0,0,0.6), 0 0 60px rgba(245,215,110,0.15);
  }
  .welcome-card .icon { font-size: 52px; margin-bottom: 10px; }
  .welcome-card h2 { font-size: 24px; color: #f5d76e; margin-bottom: 10px; }
  .welcome-card p { color: #b8c2cf; font-size: 14px; line-height: 1.6; margin-bottom: 8px; }
  .welcome-card .small { color: #7d879a; font-size: 12px; margin-top: 12px; margin-bottom: 20px; }
  .welcome-card .bonus {
    display: inline-block; font-size: 12px; color: #f5d76e;
    background: rgba(245,215,110,0.15);
    border: 1px solid rgba(245,215,110,0.4);
    padding: 6px 14px; border-radius: 8px; margin: 6px 0;
    font-weight: 700;
  }
  .welcome-card .btn {
    padding: 12px 34px; border-radius: 12px; border: none;
    font-size: 15px; font-weight: 700; cursor: pointer;
    background: linear-gradient(135deg, #f5d76e, #f8b500);
    color: #1b2735;
  }

  .stats { display: flex; gap: 10px; margin-bottom: 16px; flex-wrap: wrap; }
  .stat {
    flex: 1; min-width: 120px;
    background: rgba(255,255,255,0.05);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 12px; padding: 10px; text-align: center;
  }
  .stat .label { font-size: 10px; color: #8a94a6; text-transform: uppercase; letter-spacing: 1px; }
  .stat .value { font-size: 18px; font-weight: 700; color: #f5d76e; margin-top: 3px; }
  .stat .value.money { color: #6ee7a8; }

  .region-select { display: flex; gap: 8px; margin-bottom: 16px; flex-wrap: wrap; }
  .region-btn {
    flex: 1; min-width: 110px; padding: 9px;
    border-radius: 10px;
    border: 1px solid rgba(255,255,255,0.15);
    background: rgba(255,255,255,0.04);
    color: #cdd5e0; cursor: pointer; font-size: 12px;
    transition: all .2s; text-align: center;
  }
  .region-btn:hover { background: rgba(255,255,255,0.1); }
  .region-btn.active {
    background: linear-gradient(135deg, #f5d76e, #f8b500);
    color: #1b2735; font-weight: 700; border-color: transparent;
  }
  .region-btn .flag { font-size: 18px; display: block; margin-bottom: 1px; }

  .generator {
    background: rgba(255,255,255,0.05);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 16px; padding: 22px; text-align: center; margin-bottom: 16px;
    position: relative;
    overflow: hidden;
  }

  .flash {
    position: absolute; inset: 0;
    pointer-events: none; opacity: 0; z-index: 5;
  }
  .flash.active { animation: flashAnim 0.6s ease-out; }
  @keyframes flashAnim {
    0%   { opacity: 0.9; }
    100% { opacity: 0; }
  }
  .flash.common    { background: radial-gradient(circle, rgba(255,255,255,0.4) 0%, transparent 70%); }
  .flash.rare      { background: radial-gradient(circle, rgba(59,130,246,0.7) 0%, transparent 70%); }
  .flash.epic      { background: radial-gradient(circle, rgba(168,85,247,0.8) 0%, transparent 70%); }
  .flash.mythic    { background: radial-gradient(circle, rgba(249,115,22,0.9) 0%, transparent 70%); }
  .flash.legendary { background: radial-gradient(circle, rgba(239,68,68,1) 0%, transparent 70%); }
  .flash.rarest    { background: radial-gradient(circle, rgba(255,215,0,1) 0%, transparent 70%); }
  .flash.incredible{ background: radial-gradient(circle, rgba(0,255,255,1) 0%, transparent 70%); }
  .flash.absolute  { background: radial-gradient(circle, rgba(255,255,255,1) 0%, transparent 70%); }
  .flash.god       { background: radial-gradient(circle, rgba(255,0,255,1) 0%, rgba(255,215,0,0.5) 50%, transparent 70%); }
  .flash.candy     { background: radial-gradient(circle, rgba(255,107,203,1) 0%, transparent 70%); }

  .plate-wrap { display: inline-block; margin-bottom: 10px; }
  .plate {
    display: flex; background: #fff;
    border: 3px solid #111; border-radius: 6px;
    padding: 6px 0;
    font-family: 'Arial Narrow', 'Roboto Condensed', 'Segoe UI', sans-serif;
    font-weight: 900; color: #111;
    box-shadow: 0 8px 24px rgba(0,0,0,0.5), inset 0 -2px 0 rgba(0,0,0,0.15);
    transition: all .3s; user-select: none; min-width: 400px;
  }
  .plate-main {
    display: flex; align-items: center;
    padding: 0 14px;
    border-right: 2px solid #111;
    flex: 1; justify-content: center;
  }
  .plate-region {
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    padding: 0 10px; min-width: 78px;
  }
  .plate-region .code { font-size: 26px; line-height: 1; font-weight: 900; }
  .plate-region .country { font-size: 11px; font-weight: 700; letter-spacing: 1px; margin-top: 1px; }
  .plate-region .flag { font-size: 16px; line-height: 1; margin-top: 1px; }
  .plate .l { font-size: 32px; line-height: 1; }
  .plate .d { font-size: 36px; line-height: 1; letter-spacing: 1px; }
  .plate .gap { width: 6px; }

  .plate.plate-animate { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2); }
  @keyframes plateIn {
    0%   { opacity: 0; transform: scale(0.4) rotateY(90deg); }
    50%  { opacity: 1; transform: scale(1.15) rotateY(0deg); }
    75%  { transform: scale(0.95); }
    100% { transform: scale(1); }
  }
  .plate.plate-rare-anim    { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2); }
  .plate.plate-epic-anim    { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2), epicShake 0.5s ease-out 0.7s; }
  .plate.plate-mythic-anim  { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2), epicShake 0.7s ease-out 0.7s; }
  .plate.plate-legendary-anim { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2), legendaryShake 1s ease-out 0.7s; }
  .plate.plate-rarest-anim    { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2), legendaryShake 1.2s ease-out 0.7s; }
  .plate.plate-incredible-anim { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2), legendaryShake 1.5s ease-out 0.7s; }
  .plate.plate-absolute-anim   { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2), legendaryShake 2s ease-out 0.7s; }
  .plate.plate-god-anim        { animation: plateIn 0.9s cubic-bezier(.2,.9,.3,1.2), godShake 2.5s ease-out 0.7s; }
  .plate.plate-candy-anim   { animation: plateIn 0.7s cubic-bezier(.2,.9,.3,1.2), candySpin 0.8s ease-out 0.7s; }

  @keyframes epicShake {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-4px); }
    75% { transform: translateX(4px); }
  }
  @keyframes legendaryShake {
    0%, 100% { transform: translateX(0); }
    20% { transform: translateX(-6px); }
    40% { transform: translateX(6px); }
    60% { transform: translateX(-4px); }
    80% { transform: translateX(4px); }
  }
  @keyframes godShake {
    0%, 100% { transform: translateX(0) scale(1); }
    10% { transform: translateX(-10px) scale(1.02); }
    30% { transform: translateX(10px) scale(1.05); }
    50% { transform: translateX(-8px) scale(1.02); }
    70% { transform: translateX(8px) scale(1.03); }
    90% { transform: translateX(-4px) scale(1); }
  }
  @keyframes candySpin {
    0%   { transform: rotate(0deg) scale(1); }
    50%  { transform: rotate(180deg) scale(1.1); }
    100% { transform: rotate(360deg) scale(1); }
  }

  .plate.common    { border-color: #444; box-shadow: 0 0 0 2px rgba(255,255,255,0.08), 0 8px 24px rgba(0,0,0,0.5); }
  .plate.rare      { border-color: #3b82f6; box-shadow: 0 0 24px rgba(59,130,246,0.7), 0 8px 24px rgba(0,0,0,0.5); }
  .plate.epic      { border-color: #a855f7; box-shadow: 0 0 28px rgba(168,85,247,0.8), 0 8px 24px rgba(0,0,0,0.5); }
  .plate.mythic    { border-color: #f97316; box-shadow: 0 0 32px rgba(249,115,22,0.9), 0 8px 24px rgba(0,0,0,0.5); }
  .plate.legendary { border-color: #ef4444; box-shadow: 0 0 40px rgba(239,68,68,1), 0 0 80px rgba(239,68,68,0.5), 0 8px 24px rgba(0,0,0,0.5); animation: pulse 1.4s infinite alternate; }
  .plate.rarest    { border-color: #ffd700; box-shadow: 0 0 40px rgba(255,215,0,1), 0 0 80px rgba(255,215,0,0.6), 0 8px 24px rgba(0,0,0,0.5); animation: pulseRarest 1.4s infinite alternate; }
  .plate.incredible{ border-color: #00ffff; box-shadow: 0 0 50px rgba(0,255,255,1), 0 0 100px rgba(0,255,255,0.7), 0 8px 24px rgba(0,0,0,0.5); animation: pulseIncredible 1.2s infinite alternate; }
  .plate.absolute  { border-color: #ffffff; background: linear-gradient(135deg, #fff 0%, #e8e8ff 50%, #fff 100%); box-shadow: 0 0 60px rgba(255,255,255,1), 0 0 120px rgba(255,255,255,0.7), 0 8px 24px rgba(0,0,0,0.5); animation: pulseAbsolute 1s infinite alternate; }
  .plate.god       {
    border-color: #ff00ff;
    background: linear-gradient(135deg, #fff 0%, #ffe8ff 25%, #ffd7ff 50%, #ffe8ff 75%, #fff 100%);
    box-shadow: 0 0 60px rgba(255,0,255,1), 0 0 120px rgba(255,215,0,0.8), 0 0 200px rgba(255,0,255,0.6), 0 8px 24px rgba(0,0,0,0.5);
    animation: pulseGod 1s infinite alternate;
  }
  .plate.god .l, .plate.god .d { color: #a000a0; text-shadow: 0 0 12px rgba(255,0,255,0.7); }

  .plate.candy {
    border-color: #ff6bcb;
    background: linear-gradient(135deg, #fff 0%, #ffe8f5 50%, #fff 100%);
    box-shadow: 0 0 30px rgba(255,107,203,0.9), 0 0 60px rgba(255,107,203,0.5), 0 8px 24px rgba(0,0,0,0.5);
    animation: candyPulse 1.6s infinite alternate;
  }
  @keyframes candyPulse {
    from { box-shadow: 0 0 20px rgba(255,107,203,0.7), 0 0 40px rgba(255,107,203,0.4), 0 8px 24px rgba(0,0,0,0.5); }
    to   { box-shadow: 0 0 40px rgba(255,107,203,1),   0 0 90px rgba(255,107,203,0.7), 0 8px 24px rgba(0,0,0,0.5); }
  }
  .plate.candy .l, .plate.candy .d { color: #d6008c; }
  @keyframes pulse {
    from { box-shadow: 0 0 30px rgba(239,68,68,0.8), 0 0 60px rgba(239,68,68,0.4), 0 8px 24px rgba(0,0,0,0.5); }
    to   { box-shadow: 0 0 50px rgba(239,68,68,1),   0 0 100px rgba(239,68,68,0.7), 0 8px 24px rgba(0,0,0,0.5); }
  }
  @keyframes pulseRarest {
    from { box-shadow: 0 0 30px rgba(255,215,0,0.8), 0 0 60px rgba(255,215,0,0.4), 0 8px 24px rgba(0,0,0,0.5); }
    to   { box-shadow: 0 0 60px rgba(255,215,0,1),   0 0 120px rgba(255,215,0,0.7), 0 8px 24px rgba(0,0,0,0.5); }
  }
  @keyframes pulseIncredible {
    from { box-shadow: 0 0 40px rgba(0,255,255,0.8), 0 0 80px rgba(0,255,255,0.4), 0 8px 24px rgba(0,0,0,0.5); }
    to   { box-shadow: 0 0 80px rgba(0,255,255,1),   0 0 160px rgba(0,255,255,0.7), 0 8px 24px rgba(0,0,0,0.5); }
  }
  @keyframes pulseAbsolute {
    from { box-shadow: 0 0 50px rgba(255,255,255,0.8), 0 0 100px rgba(255,255,255,0.5), 0 8px 24px rgba(0,0,0,0.5); }
    to   { box-shadow: 0 0 100px rgba(255,255,255,1),   0 0 200px rgba(255,255,255,0.8), 0 8px 24px rgba(0,0,0,0.5); }
  }
  @keyframes pulseGod {
    from { box-shadow: 0 0 50px rgba(255,0,255,0.8), 0 0 100px rgba(255,215,0,0.6), 0 0 180px rgba(255,0,255,0.4), 0 8px 24px rgba(0,0,0,0.5); }
    to   { box-shadow: 0 0 100px rgba(255,0,255,1),  0 0 200px rgba(255,215,0,0.9), 0 0 300px rgba(255,0,255,0.7), 0 8px 24px rgba(0,0,0,0.5); }
  }
  .plate.expensive { animation: expensivePulse 1s infinite alternate; border-color: #ffd700; }
  @keyframes expensivePulse {
    from { box-shadow: 0 0 30px rgba(255,215,0,0.9), 0 8px 24px rgba(0,0,0,0.5); }
    to   { box-shadow: 0 0 60px rgba(255,215,0,1), 0 8px 24px rgba(0,0,0,0.5); }
  }

  .plate-info { font-size: 13px; color: #8a94a6; margin-bottom: 4px; min-height: 18px; }
  .plate-info.info-animate { animation: infoIn 0.5s ease 0.5s backwards; }
  @keyframes infoIn {
    from { opacity: 0; transform: translateY(10px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  .plate-value { font-size: 22px; font-weight: 700; color: #6ee7a8; margin-bottom: 14px; }
  .plate-value.value-animate { animation: valueIn 0.6s ease 0.7s backwards; }
  @keyframes valueIn {
    from { opacity: 0; transform: scale(0.5); }
    to   { opacity: 1; transform: scale(1); }
  }
  .plate-value .mult { font-size: 13px; color: #f5d76e; }
  .plate-value.expensive-value { color: #ffd700; }
  .plate-value .rarity-tag {
    display: inline-block; font-size: 11px; padding: 2px 8px;
    border-radius: 6px; margin-left: 6px; vertical-align: middle;
    font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px;
  }
  .rarity-tag.common    { background:#444; color:#ddd; }
  .rarity-tag.rare      { background:#3b82f6; color:#fff; }
  .rarity-tag.epic      { background:#a855f7; color:#fff; }
  .rarity-tag.mythic    { background:#f97316; color:#fff; }
  .rarity-tag.legendary { background:#ef4444; color:#fff; }
  .rarity-tag.rarest    { background:#ffd700; color:#1b2735; }
  .rarity-tag.incredible{ background:#00ffff; color:#1b2735; }
  .rarity-tag.absolute  { background:#ffffff; color:#1b2735; }
  .rarity-tag.god       { background: linear-gradient(135deg, #ff00ff, #ffd700); color:#fff; }
  .rarity-tag.candy     { background:#ff6bcb; color:#fff; }

  .buttons { display: flex; gap: 10px; margin-bottom: 12px; }
  .btn {
    flex: 1; padding: 13px; border-radius: 12px; border: none;
    font-size: 14px; font-weight: 700; cursor: pointer; transition: all .2s;
  }
  .btn:disabled { opacity: .4; cursor: not-allowed; }
  .btn-generate { background: linear-gradient(135deg, #6ee7a8, #34d399); color: #052e1b; }
  .btn-sell { background: linear-gradient(135deg, #f5d76e, #f8b500); color: #1b2735; }

  .case-select {
    background: rgba(0,0,0,0.25);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 14px; padding: 12px 14px;
  }
  .case-select .case-title {
    font-size: 11px; color: #8a94a6;
    text-transform: uppercase; letter-spacing: 1px;
    text-align: center; margin-bottom: 10px;
  }
  .case-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
    gap: 8px;
  }
  .case-item {
    background: rgba(255,255,255,0.04);
    border: 2px solid rgba(255,255,255,0.15);
    border-radius: 10px; padding: 10px 8px;
    text-align: center; cursor: pointer; transition: all .2s;
  }
  .case-item.disabled { opacity: 0.4; cursor: not-allowed; }
  .case-item.active { border-color: #f5d76e; box-shadow: 0 0 16px rgba(245,215,110,0.5); }
  .case-item .case-icon { font-size: 22px; margin-bottom: 4px; }
  .case-item .case-name { font-size: 12px; font-weight: 700; margin-bottom: 2px; }
  .case-item .case-price { font-size: 10px; color: #f5d76e; font-weight: 700; }
  .case-item .case-locked { font-size: 9px; color: #ff6b6b; margin-top: 2px; font-weight: 700; }
  .case-item.c-common    { border-color: #555; } .case-item.c-common    .case-name { color: #ccc; }
  .case-item.c-rare      { border-color: #3b82f6; } .case-item.c-rare      .case-name { color: #60a5fa; }
  .case-item.c-epic      { border-color: #a855f7; } .case-item.c-epic      .case-name { color: #c084fc; }
  .case-item.c-mythic    { border-color: #f97316; } .case-item.c-mythic    .case-name { color: #fb923c; }
  .case-item.c-legendary { border-color: #ef4444; } .case-item.c-legendary .case-name { color: #ff5555; }

  .log {
    background: rgba(0,0,0,0.3);
    border: 1px solid rgba(255,255,255,0.08);
    border-radius: 12px; padding: 10px 14px;
    max-height: 150px; overflow-y: auto;
    font-size: 12px; line-height: 1.7;
  }
  .log-item { border-bottom: 1px solid rgba(255,255,255,0.05); padding: 2px 0; }
  .log-item:last-child { border-bottom: none; }
  .log-item .green { color: #6ee7a8; }
  .log-item .gold { color: #f5d76e; }
  .log-item .red { color: #ff6b6b; }
  .log-item .purple { color: #c084fc; }
  .log-item .orange { color: #fb923c; }
  .log-item .pink { color: #ff8fdb; }
  .log-item .blue { color: #60a5fa; }
  .log-item .cyan { color: #00ffff; }
  .log-item .white { color: #ffffff; }
  .log-item .magenta { color: #ff00ff; }

  .reset-btn {
    display: block; margin: 20px auto 0;
    padding: 8px 16px; border-radius: 8px;
    border: 1px solid rgba(255,107,107,0.4);
    background: transparent; color: #ff6b6b;
    font-size: 11px; cursor: pointer;
  }

  .confirm-overlay {
    position: fixed; inset: 0;
    background: rgba(0,0,0,0.85);
    backdrop-filter: blur(6px);
    display: none; align-items: center; justify-content: center;
    z-index: 1800; padding: 20px;
  }
  .confirm-overlay.show { display: flex; }
  .confirm-card {
    background: linear-gradient(160deg, rgba(40,30,15,0.98), rgba(20,15,8,0.99));
    border: 2px solid rgba(255,215,0,0.6);
    border-radius: 18px; padding: 28px 30px;
    max-width: 460px; width: 100%; text-align: center;
  }
  .confirm-card .warn-icon { font-size: 44px; margin-bottom: 8px; }
  .confirm-card h3 { color: #ffd700; font-size: 19px; margin-bottom: 10px; }
  .confirm-card .warn-text { color: #d4c88a; font-size: 13px; line-height: 1.6; margin-bottom: 8px; }
  .confirm-card .warn-value { font-size: 22px; font-weight: 900; color: #ffd700; margin: 10px 0 6px; }
  .confirm-card .warn-plate {
    font-family: 'Arial Narrow', monospace;
    font-weight: 900; color: #fff;
    background: #222; border: 2px solid #ffd700;
    padding: 4px 12px; border-radius: 6px;
    display: inline-block; margin-bottom: 12px;
    font-size: 16px; letter-spacing: 1px;
  }
  .confirm-card .confirm-actions { display: flex; gap: 10px; margin-top: 16px; }
  .confirm-card .confirm-actions button {
    flex: 1; padding: 12px; border-radius: 10px;
    border: none; cursor: pointer; font-size: 13px; font-weight: 700;
  }
  .confirm-card .btn-cancel { background: rgba(255,255,255,0.1); color: #cdd5e0; border: 1px solid rgba(255,255,255,0.2); }
  .confirm-card .btn-confirm { background: linear-gradient(135deg, #ff6b6b, #dc2626); color: #fff; }

  /* ============ СПЕЦ-ЭКРАН ============ */
  .reveal-screen {
    position: fixed; inset: 0;
    z-index: 3000;
    display: none;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: radial-gradient(ellipse at center, #0a0e1a 0%, #000 100%);
    padding: 30px 20px;
    overflow: hidden;
  }
  .reveal-screen.show { display: flex; animation: revealFadeIn 0.4s ease; }
  @keyframes revealFadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
  }

  .reveal-bg-glow {
    position: absolute; inset: 0;
    pointer-events: none;
    background:
      radial-gradient(circle at 20% 30%, rgba(59,130,246,0.10) 0%, transparent 40%),
      radial-gradient(circle at 80% 70%, rgba(168,85,247,0.10) 0%, transparent 40%),
      radial-gradient(circle at 50% 50%, rgba(239,68,68,0.08) 0%, transparent 60%);
    transition: background 0.6s ease;
  }
  .reveal-bg-glow.tension-1 {
    background:
      radial-gradient(circle at 20% 30%, rgba(59,130,246,0.2) 0%, transparent 40%),
      radial-gradient(circle at 80% 70%, rgba(168,85,247,0.2) 0%, transparent 40%),
      radial-gradient(circle at 50% 50%, rgba(239,68,68,0.15) 0%, transparent 60%);
  }
  .reveal-bg-glow.tension-2 {
    background:
      radial-gradient(circle at 20% 30%, rgba(59,130,246,0.35) 0%, transparent 40%),
      radial-gradient(circle at 80% 70%, rgba(168,85,247,0.35) 0%, transparent 40%),
      radial-gradient(circle at 50% 50%, rgba(239,68,68,0.3) 0%, transparent 60%);
  }
  .reveal-bg-glow.tension-3 {
    background:
      radial-gradient(circle at 20% 30%, rgba(255,215,0,0.4) 0%, transparent 40%),
      radial-gradient(circle at 80% 70%, rgba(255,0,255,0.4) 0%, transparent 40%),
      radial-gradient(circle at 50% 50%, rgba(239,68,68,0.5) 0%, transparent 60%);
    animation: revealBgPulse 0.6s infinite alternate;
  }
  @keyframes revealBgPulse {
    from { opacity: 0.7; }
    to   { opacity: 1; }
  }

  .reveal-content {
    position: relative;
    z-index: 2;
    width: 100%;
    max-width: 700px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .reveal-rarity-bar-wrap {
    width: 100%;
    margin-bottom: 40px;
    text-align: center;
  }
  .reveal-rarity-label {
    font-size: 12px;
    color: #8a94a6;
    text-transform: uppercase;
    letter-spacing: 3px;
    margin-bottom: 12px;
    font-weight: 700;
  }
  .reveal-rarity-bar {
    position: relative;
    height: 26px;
    background: rgba(0,0,0,0.6);
    border: 2px solid rgba(255,255,255,0.15);
    border-radius: 14px;
    overflow: hidden;
    box-shadow: inset 0 2px 8px rgba(0,0,0,0.6);
  }
  .reveal-rarity-marks {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }
  .reveal-rarity-mark {
    position: absolute;
    top: 0; bottom: 0;
    width: 1px;
    background: rgba(255,255,255,0.15);
  }
  .reveal-rarity-fill {
    position: absolute;
    left: 0; top: 0; bottom: 0;
    width: 0%;
    background: linear-gradient(90deg, #444, #6ee7a8, #3b82f6, #a855f7, #f97316, #ef4444, #ffd700, #00ffff, #ffffff, #ff00ff);
    border-radius: 12px;
    box-shadow: 0 0 20px rgba(255,255,255,0.3);
    transition: width 0.25s ease;
  }
  .reveal-rarity-text {
    font-size: 14px;
    color: #8a94a6;
    margin-top: 12px;
    font-weight: 700;
    letter-spacing: 2px;
    min-height: 24px;
    text-transform: uppercase;
    transition: all 0.3s ease;
  }
  .reveal-rarity-text.done {
    color: #f5d76e;
    font-size: 17px;
    animation: rarityTextPop 0.5s ease;
  }
  .reveal-rarity-text.done.rarest     { color: #ffd700; text-shadow: 0 0 12px rgba(255,215,0,0.7); }
  .reveal-rarity-text.done.incredible { color: #00ffff; text-shadow: 0 0 12px rgba(0,255,255,0.7); }
  .reveal-rarity-text.done.absolute   { color: #ffffff; text-shadow: 0 0 16px rgba(255,255,255,1); }
  .reveal-rarity-text.done.god        { color: #ff00ff; text-shadow: 0 0 20px rgba(255,0,255,1), 0 0 40px rgba(255,215,0,0.8); animation: godText 0.6s infinite alternate; }
  @keyframes godText {
    from { text-shadow: 0 0 16px rgba(255,0,255,0.8), 0 0 32px rgba(255,215,0,0.5); }
    to   { text-shadow: 0 0 30px rgba(255,0,255,1),   0 0 60px rgba(255,215,0,0.9); }
  }
  @keyframes rarityTextPop {
    0% { transform: scale(0.8); }
    50% { transform: scale(1.2); }
    100% { transform: scale(1); }
  }

  .reveal-plate-wrap {
    display: flex;
    justify-content: center;
    margin-bottom: 30px;
  }
  .reveal-plate {
    transform: scale(1);
    animation: revealPlateIn 0.7s cubic-bezier(.2,.9,.3,1.2);
  }
  @keyframes revealPlateIn {
    0%   { opacity: 0; transform: scale(0.3) rotateY(180deg); }
    60%  { opacity: 1; transform: scale(1.1) rotateY(0deg); }
    80%  { transform: scale(0.95); }
    100% { transform: scale(1); }
  }

  .reveal-price-wrap {
    text-align: center;
    opacity: 0;
    transform: translateY(30px);
    transition: opacity 0.5s ease, transform 0.5s cubic-bezier(.2,.9,.3,1.2);
    margin-bottom: 30px;
    min-height: 120px;
  }
  .reveal-price-wrap.show {
    opacity: 1;
    transform: translateY(0);
  }
  .reveal-price-label {
    font-size: 12px;
    color: #8a94a6;
    text-transform: uppercase;
    letter-spacing: 3px;
    margin-bottom: 8px;
    font-weight: 700;
  }
  .reveal-price-value {
    font-size: 46px;
    font-weight: 900;
    color: #ffd700;
    text-shadow: 0 0 20px rgba(255,215,0,0.7), 0 0 40px rgba(255,215,0,0.4);
    animation: priceGlow 1.4s infinite alternate;
    word-break: break-word;
    line-height: 1.1;
  }
  @keyframes priceGlow {
    from { text-shadow: 0 0 20px rgba(255,215,0,0.6), 0 0 40px rgba(255,215,0,0.3); }
    to   { text-shadow: 0 0 30px rgba(255,215,0,1),   0 0 60px rgba(255,215,0,0.6); }
  }
  .reveal-price-value.god {
    color: #ff00ff;
    text-shadow: 0 0 30px rgba(255,0,255,1), 0 0 60px rgba(255,215,0,0.9), 0 0 100px rgba(255,0,255,0.7);
  }
  .reveal-price-tags {
    font-size: 14px;
    color: #b8c2cf;
    margin-top: 10px;
    min-height: 20px;
  }
  .reveal-price-tags .tag {
    display: inline-block;
    background: rgba(255,255,255,0.08);
    border: 1px solid rgba(255,255,255,0.15);
    padding: 3px 10px;
    border-radius: 6px;
    font-size: 12px;
    margin: 2px;
  }

  .reveal-actions {
    display: flex;
    gap: 12px;
    width: 100%;
    max-width: 460px;
    opacity: 0;
    transform: translateY(20px);
    transition: opacity 0.5s ease, transform 0.5s ease;
  }
  .reveal-actions.show {
    opacity: 1;
    transform: translateY(0);
  }
  .reveal-btn {
    flex: 1;
    padding: 16px;
    border-radius: 14px;
    border: none;
    font-size: 15px;
    font-weight: 800;
    cursor: pointer;
    transition: all .2s;
    letter-spacing: 0.5px;
  }
  .reveal-btn.sell {
    background: linear-gradient(135deg, #f5d76e, #f8b500);
    color: #1b2735;
    box-shadow: 0 6px 20px rgba(245,215,110,0.5);
  }
  .reveal-btn.sell:hover { transform: translateY(-2px); filter: brightness(1.1); }
  .reveal-btn.skip {
    background: transparent;
    color: #cdd5e0;
    border: 2px solid rgba(255,255,255,0.2);
  }
  .reveal-btn.skip:hover { background: rgba(255,255,255,0.08); }

  /* Админ */
  .admin-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.75); display: none; align-items: center; justify-content: center; z-index: 2000; padding: 20px; }
  .admin-overlay.show { display: flex; }
  .admin-panel {
    background: linear-gradient(160deg, rgba(20,15,35,0.98), rgba(10,8,20,0.99));
    border: 1px solid rgba(168,85,247,0.5);
    border-radius: 18px; padding: 24px 26px;
    max-width: 620px; width: 100%;
    max-height: 90vh; overflow-y: auto;
  }
  .admin-panel h2 { text-align: center; color: #c084fc; font-size: 20px; margin-bottom: 4px; }
  .admin-panel .admin-hint { text-align: center; color: #7d879a; font-size: 11px; margin-bottom: 18px; }
  .admin-section-title { font-size: 11px; color: #8a94a6; text-transform: uppercase; letter-spacing: 1px; margin: 14px 0 8px; }
  .admin-event { display: flex; justify-content: space-between; align-items: center; background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.08); border-radius: 12px; padding: 12px 14px; margin-bottom: 8px; gap: 10px; }
  .admin-event .info { flex: 1; min-width: 0; }
  .admin-event .info .name { font-size: 13px; font-weight: 700; }
  .admin-event .info .desc { font-size: 11px; color: #8a94a6; margin-top: 2px; }
  .admin-event.candy .info .name { color: #ff8fdb; }
  .admin-event.lucky .info .name { color: #6ee7a8; }
  .admin-event.mega  .info .name { color: #c084fc; }
  .admin-event .controls { display: flex; gap: 6px; flex-shrink: 0; align-items: center; }
  .admin-event select { background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.15); border-radius: 8px; color: #e8e8e8; font-size: 11px; padding: 6px 8px; cursor: pointer; outline: none; }
  .admin-btn { padding: 6px 12px; border-radius: 8px; border: none; cursor: pointer; font-size: 11px; font-weight: 700; }
  .admin-btn.start { background: linear-gradient(135deg, #a855f7, #ec4899); color: #fff; }
  .admin-btn.stop { background: rgba(255,107,107,0.2); color: #ff6b6b; border: 1px solid rgba(255,107,107,0.4); }
  .admin-close { display: block; margin: 16px auto 0; padding: 10px 24px; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2); background: transparent; color: #cdd5e0; font-size: 12px; cursor: pointer; }
  .admin-give { background: rgba(245,215,110,0.08); border: 1px solid rgba(245,215,110,0.3); border-radius: 12px; padding: 12px 14px; margin-bottom: 8px; }
  .admin-give .row { display: flex; gap: 8px; align-items: center; }
  .admin-give input { flex: 1; padding: 8px 10px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.15); background: rgba(0,0,0,0.4); color: #e8e8e8; font-size: 12px; outline: none; }
  .admin-give button { padding: 8px 14px; border-radius: 8px; border: none; cursor: pointer; background: linear-gradient(135deg, #f5d76e, #f8b500); color: #1b2735; font-weight: 700; font-size: 12px; }
  .admin-broadcast-block { background: rgba(168,85,247,0.08); border: 1px solid rgba(168,85,247,0.35); border-radius: 12px; padding: 12px 14px; margin-bottom: 8px; }
  .admin-broadcast-block input[type="text"] { width: 100%; padding: 9px 12px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.15); background: rgba(0,0,0,0.4); color: #e8e8e8; font-size: 13px; outline: none; margin-bottom: 8px; }
  .admin-broadcast-block .row { display: flex; gap: 8px; flex-wrap: wrap; }
  .admin-broadcast-block select { flex: 1; min-width: 110px; background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.15); border-radius: 8px; color: #e8e8e8; font-size: 12px; padding: 8px 10px; outline: none; cursor: pointer; }
  .admin-broadcast-block .broadcast-btn { flex: 1; min-width: 120px; padding: 9px 16px; border-radius: 8px; border: none; cursor: pointer; background: linear-gradient(135deg, #a855f7, #ec4899); color: #fff; font-weight: 700; font-size: 12px; }
  .admin-slider-block { background: rgba(110,231,168,0.08); border: 1px solid rgba(110,231,168,0.35); border-radius: 12px; padding: 12px 14px; margin-bottom: 8px; }
  .admin-slider-block .slider-label { display: flex; justify-content: space-between; align-items: center; font-size: 12px; font-weight: 700; color: #6ee7a8; margin-bottom: 8px; }
  .admin-slider-block .slider-label .val { background: rgba(0,0,0,0.4); padding: 2px 10px; border-radius: 6px; font-size: 13px; color: #f5d76e; }
  .admin-slider-block input[type="range"] { width: 100%; -webkit-appearance: none; height: 8px; background: rgba(0,0,0,0.5); border-radius: 4px; outline: none; }
  .admin-slider-block input[type="range"]::-webkit-slider-thumb { -webkit-appearance: none; width: 20px; height: 20px; border-radius: 50%; background: linear-gradient(135deg, #6ee7a8, #34d399); cursor: pointer; }
  .mult-btns { display: flex; gap: 8px; flex-wrap: wrap; }
  .mult-btn { flex: 1; min-width: 70px; padding: 10px 8px; border-radius: 10px; border: 2px solid rgba(245,215,110,0.4); background: rgba(0,0,0,0.3); color: #f5d76e; font-size: 14px; font-weight: 900; cursor: pointer; }
  .mult-btn.active { background: linear-gradient(135deg, #f5d76e, #f8b500); color: #1b2735; border-color: #f5d76e; }
  .admin-broadcast { position: fixed; top: 20px; left: 50%; transform: translateX(-50%) translateY(-120%); z-index: 1500; max-width: 90%; padding: 14px 28px; border-radius: 14px; font-size: 16px; font-weight: 700; text-align: center; border: 2px solid rgba(168,85,247,0.7); background: linear-gradient(160deg, rgba(40,20,60,0.98), rgba(20,10,35,0.99)); color: #fff; transition: transform .5s; }
  .admin-broadcast.show { transform: translateX(-50%) translateY(0); }
  .admin-broadcast .badge { display: inline-block; background: linear-gradient(135deg, #a855f7, #ec4899); color: #fff; font-size: 10px; font-weight: 900; padding: 3px 10px; border-radius: 20px; letter-spacing: 1.5px; margin-right: 10px; }
  .admin-broadcast.color-gold { border-color: rgba(245,215,110,0.8); background: linear-gradient(160deg, rgba(60,45,15,0.98), rgba(30,22,8,0.99)); }
  .admin-broadcast.color-green { border-color: rgba(110,231,168,0.8); background: linear-gradient(160deg, rgba(15,50,35,0.98), rgba(8,25,18,0.99)); }
  .admin-broadcast.color-red { border-color: rgba(239,68,68,0.8); background: linear-gradient(160deg, rgba(60,20,20,0.98), rgba(30,10,10,0.99)); }
  .admin-broadcast.color-blue { border-color: rgba(59,130,246,0.8); background: linear-gradient(160deg, rgba(15,30,60,0.98), rgba(8,15,30,0.99)); }
  .admin-pass-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.85); backdrop-filter: blur(6px); display: none; align-items: center; justify-content: center; z-index: 2500; padding: 20px; }
  .admin-pass-overlay.show { display: flex; }
  .admin-pass-card { background: linear-gradient(160deg, rgba(30,20,50,0.98), rgba(15,10,30,0.99)); border: 2px solid rgba(168,85,247,0.6); border-radius: 18px; padding: 28px 30px; max-width: 380px; width: 100%; text-align: center; }
  .admin-pass-card .lock-icon { font-size: 40px; margin-bottom: 8px; }
  .admin-pass-card h3 { color: #c084fc; font-size: 18px; margin-bottom: 14px; }
  .admin-pass-card input { width: 100%; padding: 12px 14px; border-radius: 10px; border: 1px solid rgba(168,85,247,0.4); background: rgba(0,0,0,0.5); color: #e8e8e8; font-size: 14px; outline: none; text-align: center; letter-spacing: 2px; margin-bottom: 12px; }
  .admin-pass-card .error { color: #ff6b6b; font-size: 12px; min-height: 16px; margin-bottom: 8px; }
  .admin-pass-card .actions { display: flex; gap: 8px; }
  .admin-pass-card button { flex: 1; padding: 10px; border-radius: 10px; border: none; cursor: pointer; font-size: 13px; font-weight: 700; }
  .admin-pass-card .btn-cancel { background: rgba(255,255,255,0.1); color: #cdd5e0; border: 1px solid rgba(255,255,255,0.2); }
  .admin-pass-card .btn-ok { background: linear-gradient(135deg, #a855f7, #ec4899); color: #fff; }

  /* Блок ручного номера в админке */
  .admin-manual-block {
    background: rgba(255,215,0,0.08);
    border: 1px solid rgba(255,215,0,0.4);
    border-radius: 12px;
    padding: 12px 14px;
    margin-bottom: 8px;
  }
  .admin-manual-block .manual-title {
    color: #ffd700;
    font-size: 12px;
    font-weight: 700;
    margin-bottom: 8px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .admin-manual-block .manual-badge {
    font-size: 10px;
    background: rgba(255,215,0,0.2);
    color: #ffd700;
    padding: 2px 8px;
    border-radius: 10px;
    display: none;
  }
  .admin-manual-block .manual-badge.active {
    display: inline-block;
    animation: manualBadgePulse 1s infinite alternate;
  }
  @keyframes manualBadgePulse {
    from { background: rgba(255,215,0,0.2); }
    to   { background: rgba(255,215,0,0.5); }
  }
  .admin-manual-block .manual-row {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
    margin-bottom: 8px;
  }
  .admin-manual-block .manual-row select,
  .admin-manual-block .manual-row input {
    flex: 1;
    min-width: 100px;
    padding: 8px 10px;
    border-radius: 8px;
    border: 1px solid rgba(255,255,255,0.15);
    background: rgba(0,0,0,0.4);
    color: #e8e8e8;
    font-size: 12px;
    outline: none;
  }
  .admin-manual-block .manual-btns {
    display: flex;
    gap: 6px;
  }
  .admin-manual-block .manual-btns button {
    flex: 1;
    padding: 8px;
    border-radius: 8px;
    border: none;
    cursor: pointer;
    font-size: 12px;
    font-weight: 700;
  }
  .admin-manual-block .btn-set {
    background: linear-gradient(135deg, #ffd700, #f8b500);
    color: #1b2735;
  }
  .admin-manual-block .btn-clear {
    background: rgba(255,107,107,0.2);
    color: #ff6b6b;
    border: 1px solid rgba(255,107,107,0.4);
  }
</style>
</head>
<body>

<div class="admin-broadcast" id="adminBroadcast">
  <span class="badge">АДМИН</span>
  <span class="text" id="adminBroadcastText"></span>
</div>

<div class="welcome-overlay" id="welcomeOverlay">
  <div class="welcome-card">
    <div class="icon">🚗</div>
    <h2>Приветствуем вас в Нумерологе!</h2>
    <p>Генерируй госномера за деньги и продавай их дороже.</p>
    <p>Большинство номеров не окупают кейс — прибыль только с редких дропов.</p>
    <div class="bonus">🎁 Стартовый капитал: 100 000 ₽</div>
    <p class="small">9 редкостей · цены от 100 ₽ до 99 000 000 000 ₽</p>
    <button class="btn" id="welcomeStart">Начать игру</button>
  </div>
</div>

<div class="admin-pass-overlay" id="adminPassOverlay">
  <div class="admin-pass-card">
    <div class="lock-icon">🔒</div>
    <h3>Введите пароль администратора</h3>
    <input type="password" id="adminPassInput" placeholder="••••••••" autocomplete="off">
    <div class="error" id="adminPassError"></div>
    <div class="actions">
      <button class="btn-cancel" id="adminPassCancel">Отмена</button>
      <button class="btn-ok" id="adminPassOk">Войти</button>
    </div>
  </div>
</div>

<div class="reveal-screen" id="revealScreen">
  <div class="reveal-bg-glow" id="revealBgGlow"></div>

  <div class="reveal-content">
    <div class="reveal-rarity-bar-wrap">
      <div class="reveal-rarity-label">Оценка редкости</div>
      <div class="reveal-rarity-bar">
        <div class="reveal-rarity-marks" id="revealRarityMarks"></div>
        <div class="reveal-rarity-fill" id="revealRarityFill"></div>
      </div>
      <div class="reveal-rarity-text" id="revealRarityText">Сканирование...</div>
    </div>

    <div class="reveal-plate-wrap">
      <div class="plate reveal-plate" id="revealPlate">
        <div class="plate-main" id="revealPlateMain">
          <span class="l">А</span><span class="gap"></span>
          <span class="d">000</span><span class="gap"></span>
          <span class="l">АА</span>
        </div>
        <div class="plate-region">
          <div class="code" id="revealPlateCode">77</div>
          <div class="country" id="revealPlateCountry">RUS</div>
          <div class="flag" id="revealPlateFlag">🇷🇺</div>
        </div>
      </div>
    </div>

    <div class="reveal-price-wrap" id="revealPriceWrap">
      <div class="reveal-price-label">Оценка стоимости</div>
      <div class="reveal-price-value" id="revealPriceValue">—</div>
      <div class="reveal-price-tags" id="revealPriceTags"></div>
    </div>

    <div class="reveal-actions" id="revealActions">
      <button class="reveal-btn skip" id="revealSkip">🎲 Крутить дальше</button>
      <button class="reveal-btn sell" id="revealSell">💰 Продать за <span id="revealSellAmount">—</span></button>
    </div>
  </div>
</div>

<div class="confirm-overlay" id="confirmOverlay">
  <div class="confirm-card">
    <div class="warn-icon">⚠️</div>
    <h3>У вас дорогой номер!</h3>
    <div class="warn-text">Текущий номер стоит больше 100 миллионов рублей. Вы уверены, что хотите прокрутить его, не продав?</div>
    <div class="warn-plate" id="confirmPlate">—</div>
    <div class="warn-value" id="confirmValue">—</div>
    <div class="confirm-actions">
      <button class="btn-cancel" id="confirmCancel">Отмена (продать)</button>
      <button class="btn-confirm" id="confirmGo">Да, прокрутить</button>
    </div>
  </div>
</div>

<div class="admin-overlay" id="adminOverlay">
  <div class="admin-panel">
    <h2>🛠 Админ-панель</h2>
    <p class="admin-hint">Ctrl+Shift+A — открыть/закрыть · пароль обязателен</p>

    <div class="admin-section-title">Следующий номер вручную</div>
    <div class="admin-manual-block">
      <div class="manual-title">
        <span>⚙️ Задать параметры следующему номеру</span>
        <span class="manual-badge" id="manualBadge">АКТИВНО</span>
      </div>
      <div class="manual-row">
        <select id="manualRarity">
          <option value="">— Редкость —</option>
          <option value="common">Обычный</option>
          <option value="rare">Редкий</option>
          <option value="epic">Эпический</option>
          <option value="mythic">Мифический</option>
          <option value="legendary">Легендарный</option>
          <option value="rarest">Редчайший</option>
          <option value="incredible">Невероятный</option>
          <option value="absolute">Абсолютный</option>
          <option value="god">БОГ</option>
        </select>
        <input type="number" id="manualPrice" placeholder="Цена (или пусто — по редкости)">
      </div>
      <div class="manual-btns">
        <button class="btn-set" id="manualSetBtn">✓ Установить</button>
        <button class="btn-clear" id="manualClearBtn">✕ Сбросить</button>
      </div>
    </div>

    <div class="admin-section-title">Вывести надпись</div>
    <div class="admin-broadcast-block">
      <input type="text" id="broadcastText" placeholder="Введите текст сообщения..." maxlength="120">
      <div class="row">
        <select id="broadcastColor">
          <option value="purple">🟣 Фиолетовый</option>
          <option value="gold">🟡 Золотой</option>
          <option value="green">🟢 Зелёный</option>
          <option value="red">🔴 Красный</option>
          <option value="blue">🔵 Синий</option>
        </select>
        <select id="broadcastDur">
          <option value="5">5 сек</option>
          <option value="10" selected>10 сек</option>
          <option value="20">20 сек</option>
          <option value="60">1 мин</option>
          <option value="0">Пока не закрою</option>
        </select>
        <button class="broadcast-btn" id="broadcastBtn">📢 Вывести</button>
      </div>
    </div>

    <div class="admin-section-title">Уровень лаки-ивента</div>
    <div class="admin-slider-block">
      <div class="slider-label">
        <span>🍀 Сила удачи</span>
        <span class="val" id="luckLevelVal">0 / 10</span>
      </div>
      <input type="range" id="luckLevelSlider" min="0" max="10" value="0" step="1">
    </div>

    <div class="admin-section-title">Множитель кэша</div>
    <div class="mult-btns" id="multBtns">
      <button class="mult-btn" data-mult="1">x1</button>
      <button class="mult-btn" data-mult="2">x2</button>
      <button class="mult-btn" data-mult="3">x3</button>
    </div>

    <div class="admin-section-title">Ивенты</div>

    <div class="admin-event candy">
      <div class="info"><div class="name">🍬 Конфетный ивент</div><div class="desc">Шанс выпадения конфетного номера</div></div>
      <div class="controls">
        <select id="candyDur"><option value="60">1 мин</option><option value="300" selected>5 мин</option><option value="600">10 мин</option><option value="1800">30 мин</option></select>
        <button class="admin-btn start" data-event="candy">Старт</button>
        <button class="admin-btn stop" data-event="candy">Стоп</button>
      </div>
    </div>

    <div class="admin-event lucky">
      <div class="info"><div class="name">🍀 Лаки ивент</div><div class="desc">Небольшой бонус к шансу лучших номеров</div></div>
      <div class="controls">
        <select id="luckyDur"><option value="60">1 мин</option><option value="300" selected>5 мин</option><option value="600">10 мин</option><option value="1800">30 мин</option></select>
        <button class="admin-btn start" data-event="lucky">Старт</button>
        <button class="admin-btn stop" data-event="lucky">Стоп</button>
      </div>
    </div>

    <div class="admin-event mega">
      <div class="info"><div class="name">🎰 Мега-дроп</div><div class="desc">Повышает шансы редких</div></div>
      <div class="controls">
        <select id="megaDur"><option value="60">1 мин</option><option value="300" selected>5 мин</option><option value="600">10 мин</option><option value="1800">30 мин</option></select>
        <button class="admin-btn start" data-event="mega">Старт</button>
        <button class="admin-btn stop" data-event="mega">Стоп</button>
      </div>
    </div>

    <div class="admin-section-title">Выдать деньги</div>
    <div class="admin-give">
      <div class="row">
        <input type="number" id="giveAmount" placeholder="Сумма ₽" value="1000000">
        <button id="giveBtn">Выдать</button>
      </div>
    </div>

    <div class="admin-section-title">Быстрые действия</div>
    <div class="admin-event">
      <div class="info"><div class="name">🧹 Очистить лог</div><div class="desc">Убрать все записи из журнала</div></div>
      <div class="controls"><button class="admin-btn stop" id="clearLogBtn">Очистить</button></div>
    </div>

    <button class="admin-close" id="adminClose">Закрыть</button>
  </div>
</div>

<div class="game" id="gameScreen" style="display:none;">
  <h1 id="gameTitle">🚗 ГАРАЖНЫЙ НУМЕРОЛОГ</h1>
  <p class="subtitle">Генерируй номера за деньги · 9 редкостей · цены от 100 ₽ до 99 000 000 000 ₽</p>

  <div class="events-banner" id="eventsBanner"></div>

  <div class="stats">
    <div class="stat"><div class="label">Баланс</div><div class="value money" id="balance">0 ₽</div></div>
    <div class="stat"><div class="label">Продано</div><div class="value" id="sold">0</div></div>
    <div class="stat"><div class="label">Лучший</div><div class="value" id="best" style="font-size:13px;">—</div></div>
    <div class="stat"><div class="label">Сгенерировано</div><div class="value" id="rolled">0</div></div>
  </div>

  <div class="region-select" id="regions"></div>

  <div class="generator">
    <div class="flash" id="flash"></div>
    <div class="plate-wrap">
      <div class="plate" id="plate">
        <div class="plate-main" id="plateMain">
          <span class="l">А</span><span class="gap"></span>
          <span class="d">000</span><span class="gap"></span>
          <span class="l">АА</span>
        </div>
        <div class="plate-region">
          <div class="code" id="plateCode">77</div>
          <div class="country" id="plateCountry">RUS</div>
          <div class="flag" id="plateFlag">🇷🇺</div>
        </div>
      </div>
    </div>
    <div class="plate-info" id="plateInfo">Выбери кейс и нажми «Сгенерировать»</div>
    <div class="plate-value" id="plateValue"></div>
    <div class="buttons">
      <button class="btn btn-generate" id="btnGen">🎲 Сгенерировать</button>
      <button class="btn btn-sell" id="btnSell" disabled>💰 Продать</button>
    </div>

    <div class="case-select">
      <div class="case-title">🎁 Выбор редкости кейса</div>
      <div class="case-grid" id="caseGrid"></div>
    </div>
  </div>

  <div class="log" id="log">
    <div class="log-item">Добро пожаловать! Выбирай регион и открывай кейсы.</div>
  </div>

  <button class="reset-btn" id="resetBtn">Сбросить прогресс</button>
</div>

<script>
(function() {
"use strict";

// ===================== КОНСТАНТЫ =====================
var START_BALANCE = 100000;
var MAX_PRICE = 99000000000;
var SPECIAL_REVEAL_THRESHOLD = 20000000;
var BEAUTIFUL_THRESHOLD = 50000000;
var EXPENSIVE_THRESHOLD = 100000000;
var ADMIN_PASSWORD = 'sims306275';

// ===================== РЕГИОНЫ =====================
var REGIONS = [
  { id:'ru_moscow', flag:'🇷🇺', name:'Россия · Москва',  type:'ru', regionCode:'77', countryCode:'RUS', basePrice:3000 },
  { id:'ru_spb',    flag:'🇷🇺', name:'Россия · СПб',     type:'ru', regionCode:'78', countryCode:'RUS', basePrice:2800 },
  { id:'ru_region', flag:'🇷🇺', name:'Россия · Регион',  type:'ru', regionCode:'99', countryCode:'RUS', basePrice:2200 },
  { id:'by',        flag:'🇧🇾', name:'Беларусь',         type:'by', regionCode:'7',  countryCode:'BY',  basePrice:2600 },
  { id:'kz',        flag:'🇰🇿', name:'Казахстан',        type:'kz', regionCode:'01', countryCode:'KZ',  basePrice:2400 },
  { id:'ua',        flag:'🇺🇦', name:'Украина',          type:'ua', regionCode:'AA', countryCode:'UA',  basePrice:2200 },
  { id:'am',        flag:'🇦🇲', name:'Армения',          type:'am', regionCode:'01', countryCode:'AM',  basePrice:2000 }
];

var CASES = [
  { id:'common',    name:'Обычный',    icon:'📦', price:1000,   minRarity:'common' },
  { id:'rare',      name:'Редкий',     icon:'🎁', price:10000,  minRarity:'rare' },
  { id:'epic',      name:'Эпический',  icon:'💜', price:75000,  minRarity:'epic' },
  { id:'mythic',    name:'Мифический', icon:'🧡', price:150000, minRarity:'mythic' },
  { id:'legendary', name:'Легендарный',icon:'👑', price:500000, minRarity:'legendary' }
];

var RARITY_ORDER = ['common', 'rare', 'epic', 'mythic', 'legendary', 'rarest', 'incredible', 'absolute', 'god'];
var RARITY_LABELS = {
  common: 'Обычный', rare: 'Редкий', epic: 'Эпический',
  mythic: 'Мифический', legendary: 'Легендарный',
  rarest: 'Редчайший', incredible: 'Невероятный',
  absolute: 'Абсолютный', god: 'БОГ',
  candy: 'Конфетный 🍬'
};

var RARITY_THRESHOLDS = {
  rarest:    75000000,
  incredible:500000000,
  absolute:  1000000000,
  god:       99000000000
};

var RARITY_MULT = {
  common:    { min: 0.05, max: 0.35 },
  rare:      { min: 1.0,  max: 3.0 },
  epic:      { min: 8,    max: 25 },
  mythic:    { min: 80,   max: 250 },
  legendary: { min: 800,  max: 3000 },
  rarest:    { min: 25000, max: 150000 },
  incredible:{ min: 150000, max: 350000 },
  absolute:  { min: 350000, max: 1500000 },
  god:       { min: 33000000, max: 33000000 }
};

var BEAUTIFUL_PATTERNS = [
  { d:'7777', l:'ААА' }, { d:'7777', l:'ВВВ' }, { d:'7777', l:'ЕЕЕ' },
  { d:'7777', l:'ККК' }, { d:'7777', l:'МММ' }, { d:'7777', l:'ННН' },
  { d:'7777', l:'ООО' }, { d:'7777', l:'РРР' }, { d:'7777', l:'ССС' },
  { d:'0001', l:'ААА' }, { d:'0007', l:'ООО' }, { d:'0007', l:'ССС' },
  { d:'1000', l:'ААА' }, { d:'7000', l:'ООО' }, { d:'7000', l:'ССС' },
  { d:'1111', l:'ААА' }, { d:'2222', l:'ВВВ' }, { d:'3333', l:'ЕЕЕ' },
  { d:'4444', l:'ККК' }, { d:'5555', l:'МММ' }, { d:'6666', l:'ННН' },
  { d:'8888', l:'ООО' }, { d:'9999', l:'РРР' }, { d:'7777', l:'777' }
];

var LETTERS = 'АВЕКМНОРСТУХ';

var state = {
  balance: 0, sold: 0, best: 0, rolled: 0,
  region: REGIONS[0],
  current: null,
  events: {},
  adminLuckLevel: 0,
  adminCashMult: 1,
  selectedCase: 'common',
  manualRarity: null,     // заданная редкость
  manualPrice: null       // заданная цена
};

// ===================== ХРАНИЛИЩЕ =====================
var Store = (function() {
  var mode = 'memory';
  var mem = {};
  function testLS() {
    try {
      var k = '__t__' + Math.random();
      window.localStorage.setItem(k, '1');
      window.localStorage.removeItem(k);
      return true;
    } catch (e) { return false; }
  }
  function testCookie() {
    try {
      document.cookie = '__t__=1; path=/';
      var ok = document.cookie.indexOf('__t__=1') !== -1;
      document.cookie = '__t__=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/';
      return ok;
    } catch (e) { return false; }
  }
  if (testLS()) mode = 'local';
  else if (testCookie()) mode = 'cookie';

  return {
    setItem: function(k, v) {
      if (mode === 'local') { try { window.localStorage.setItem(k, v); return; } catch (e) {} }
      if (mode === 'cookie') { try { document.cookie = k + '=' + encodeURIComponent(v) + '; path=/; max-age=31536000'; return; } catch (e) {} }
      mem[k] = v;
    },
    getItem: function(k) {
      if (mode === 'local') { try { return window.localStorage.getItem(k); } catch (e) {} }
      if (mode === 'cookie') {
        try {
          var m = document.cookie.match(new RegExp('(?:^|; )' + k.replace(/[.*+?^${}()|[\]\\]/g, '\\$&') + '=([^;]*)'));
          return m ? decodeURIComponent(m[1]) : null;
        } catch (e) {}
      }
      return mem[k] !== undefined ? mem[k] : null;
    },
    removeItem: function(k) {
      if (mode === 'local') { try { window.localStorage.removeItem(k); return; } catch (e) {} }
      if (mode === 'cookie') { try { document.cookie = k + '=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/'; return; } catch (e) {} }
      delete mem[k];
    }
  };
})();

var SAVE_KEY = 'numerolog_save_v11';

function save() {
  try {
    Store.setItem(SAVE_KEY, JSON.stringify({
      balance: state.balance, sold: state.sold, best: state.best, rolled: state.rolled,
      regionId: state.region.id,
      selectedCase: state.selectedCase,
      adminLuckLevel: state.adminLuckLevel,
      adminCashMult: state.adminCashMult
    }));
  } catch (e) {}
}

function load() {
  var has = false;
  try {
    var raw = Store.getItem(SAVE_KEY);
    if (raw) {
      var d = JSON.parse(raw);
      state.balance = d.balance || 0;
      state.sold = d.sold || 0;
      state.best = d.best || 0;
      state.rolled = d.rolled || 0;
      for (var i = 0; i < REGIONS.length; i++) if (REGIONS[i].id === d.regionId) state.region = REGIONS[i];
      state.selectedCase = d.selectedCase || 'common';
      state.adminLuckLevel = d.adminLuckLevel || 0;
      state.adminCashMult = d.adminCashMult || 1;
      has = true;
    } else {
      state.balance = START_BALANCE;
      save();
    }
  } catch (e) {}
  return has;
}

function randInt(min, max) { return Math.floor(Math.random() * (max - min + 1)) + min; }
function pick(arr) { return arr[randInt(0, arr.length - 1)]; }
function formatMoney(n) { return Math.round(n).toLocaleString('ru-RU') + ' ₽'; }
function escapeHtml(s) {
  return String(s).replace(/[&<>"']/g, function(c) {
    return { '&':'&amp;', '<':'&lt;', '>':'&gt;', '"':'&quot;', "'":'&#39;' }[c];
  });
}

// ===================== ИВЕНТЫ =====================
var EVENT_INFO = {
  candy: { label: '🍬 Конфетный', cls: 'candy' },
  lucky: { label: '🍀 Лаки',      cls: 'lucky' },
  mega:  { label: '🎰 Мега-дроп', cls: 'mega'  }
};

function isEventActive(name) {
  return state.events[name] && state.events[name] > Date.now();
}

function renderEventsBanner() {
  var banner = document.getElementById('eventsBanner');
  banner.innerHTML = '';
  var now = Date.now();
  for (var k in EVENT_INFO) {
    if (state.events[k] && state.events[k] > now) {
      var remaining = Math.ceil((state.events[k] - now) / 1000);
      var m = Math.floor(remaining / 60);
      var s = String(remaining % 60).padStart(2, '0');
      var chip = document.createElement('div');
      chip.className = 'event-chip ' + EVENT_INFO[k].cls;
      chip.innerHTML = EVENT_INFO[k].label + ' <span class="timer">' + m + ':' + s + '</span>';
      banner.appendChild(chip);
    }
  }
  if (state.adminCashMult > 1) {
    var chip2 = document.createElement('div');
    chip2.className = 'event-chip money';
    chip2.innerHTML = '💰 x' + state.adminCashMult + ' кэш';
    banner.appendChild(chip2);
  }
  if (state.adminLuckLevel > 0) {
    var chip3 = document.createElement('div');
    chip3.className = 'event-chip lucky';
    chip3.innerHTML = '🍀 Удача ур.' + state.adminLuckLevel;
    banner.appendChild(chip3);
  }
}

// ===================== ОЦЕНКА КОМБИНАЦИИ =====================
function evaluateCombo(digits, letters) {
  var mult = 1;
  var tags = [];
  var d = digits.split('').map(Number);
  var allSame = d.every(function(x) { return x === d[0]; });
  var isSequence = d.every(function(x, i) { return i === 0 || x === d[i-1] + 1; });
  var isDescending = d.every(function(x, i) { return i === 0 || x === d[i-1] - 1; });
  var isPalindrome = digits === digits.split('').reverse().join('');
  var hasTriple = /(\d)\1\1/.test(digits);
  var hasPair = /(\d)\1/.test(digits);
  var sum = d.reduce(function(a, b) { return a + b; }, 0);
  var luckySum = (sum === 7 || sum === 21 || sum === 13);
  var allLucky = d.every(function(x) { return [7,8,9].indexOf(x) !== -1; });
  var startsWith7 = d[0] === 7;
  var endsWith7 = d[3] === 7;
  var lettersSame = letters[0] === letters[1] && letters[1] === letters[2];
  var lettersPair = letters[0] === letters[1] || letters[1] === letters[2];

  if (allSame && d[0] === 7) { mult *= 500; tags.push('👑 7777'); }
  else if (allSame && d[0] === 0) { mult *= 250; tags.push('💠 0000'); }
  else if (allSame) { mult *= 180; tags.push('💎 Все цифры одинаковые'); }
  else if (isSequence && d[0] === 1) { mult *= 150; tags.push('🔥 1234'); }
  else if (isSequence) { mult *= 100; tags.push('📈 Последовательность'); }
  else if (isDescending) { mult *= 90; tags.push('📉 Обратный порядок'); }
  else if (hasTriple) { mult *= 60; tags.push('🎯 Триплет'); }
  else if (isPalindrome) { mult *= 50; tags.push('🔄 Палиндром'); }
  else if (allLucky) { mult *= 35; tags.push('🍀 Только 7-8-9'); }
  else if (hasPair && startsWith7 && endsWith7) { mult *= 30; tags.push('✨ 7xx7'); }
  else if (digits === '0001' || digits === '0007') { mult *= 25; tags.push('🥇 Особый'); }
  else if (digits === '1000' || digits === '7000') { mult *= 22; tags.push('💯 Круглое'); }
  else if (hasPair) { mult *= 10; tags.push('👥 Пара цифр'); }
  else if (luckySum) { mult *= 6; tags.push('🔮 Счастливая сумма'); }
  else if (startsWith7 || endsWith7) { mult *= 4; tags.push('7️⃣ Семёрка'); }

  if (lettersSame) { mult *= 7; tags.push('🔤 Одинаковые буквы'); }
  else if (lettersPair) { mult *= 3; tags.push('🔡 Пара букв'); }

  var tagStr = tags.length ? tags.join(' · ') : 'Обычный номер';
  return { mult: mult, tags: tagStr };
}

function generateBeautifulPlate(region) {
  var pattern = pick(BEAUTIFUL_PATTERNS);
  var digits = pattern.d;
  var letters;
  if (region.type === 'ru') letters = pattern.l;
  else letters = pattern.l.slice(0, 2);
  return { digits: digits, letters: letters };
}

function rarityFromPrice(price) {
  if (price >= RARITY_THRESHOLDS.god) return 'god';
  if (price >= RARITY_THRESHOLDS.absolute) return 'absolute';
  if (price >= RARITY_THRESHOLDS.incredible) return 'incredible';
  if (price >= RARITY_THRESHOLDS.rarest) return 'rarest';
  if (price >= 2400000) return 'legendary';
  if (price >= 240000) return 'mythic';
  if (price >= 24000) return 'epic';
  if (price >= 3000) return 'rare';
  return 'common';
}

function computeBasePrice(rarity, region, isCandy) {
  var r = RARITY_MULT[rarity] || RARITY_MULT.common;
  var factor = r.min + Math.random() * (r.max - r.min);
  var price = region.basePrice * factor;
  if (isCandy) price *= 5;
  return price;
}

// ===================== ОТКРЫТИЕ КЕЙСА =====================
function openCase(caseId) {
  var kase = null;
  for (var i = 0; i < CASES.length; i++) if (CASES[i].id === caseId) kase = CASES[i];
  if (!kase) return;

  if (state.balance < kase.price) {
    addLog('❌ Недостаточно денег для кейса <b>' + kase.name + '</b> (нужно ' + formatMoney(kase.price) + ')');
    return;
  }
  state.balance -= kase.price;

  var region = state.region;
  var digits, letters, combo, price, finalRarity, isCandy = false, isManual = false;

  // === ПРОВЕРКА РУЧНОЙ НАСТРОЙКИ ===
  if (state.manualRarity || state.manualPrice) {
    isManual = true;
    // Сначала генерируем красивую комбинацию, если задана редкость
    var mRar = state.manualRarity;
    var mPrice = state.manualPrice;

    if (mRar && !mPrice) {
      // Задана только редкость — цена по её диапазону
      var baseP = computeBasePrice(mRar, region, false);
      mPrice = baseP;
    } else if (!mRar && mPrice) {
      // Задана только цена — редкость вычисляется
      mRar = rarityFromPrice(mPrice);
    }

    // Генерируем номер
    var gen = generateBeautifulPlate(region);
    // Если редкость не "красивая" — можно обычный номер
    if (mRar === 'common' || mRar === 'rare' || mRar === 'epic' || mRar === 'mythic' || mRar === 'legendary') {
      // Обычные редкости могут быть и обычными номерами
      if (Math.random() < 0.5) {
        digits = String(randInt(1, 9999)).padStart(4, '0');
        if (region.type === 'ru') {
          letters = pick(LETTERS.split('')) + pick(LETTERS.split('')) + pick(LETTERS.split(''));
        } else {
          letters = String.fromCharCode(65 + randInt(0, 25)) + String.fromCharCode(65 + randInt(0, 25));
        }
      } else {
        digits = gen.digits;
        letters = gen.letters;
      }
    } else {
      // Для rarest+ — всегда красивый
      digits = gen.digits;
      letters = gen.letters;
    }

    combo = evaluateCombo(digits, letters);
    price = mPrice;

    // Сброс после применения
    state.manualRarity = null;
    state.manualPrice = null;
    updateManualBadge();

    addLog('⚙️ Админ выдал заданный номер: редкость <b>' + (RARITY_LABELS[mRar] || '—') + '</b>, цена <b>' + formatMoney(price) + '</b>');
    finalRarity = mRar;
  } else {
    // === ОБЫЧНАЯ ГЕНЕРАЦИЯ ===
    digits = String(randInt(1, 9999)).padStart(4, '0');
    if (region.type === 'ru') {
      letters = pick(LETTERS.split('')) + pick(LETTERS.split('')) + pick(LETTERS.split(''));
    } else {
      letters = String.fromCharCode(65 + randInt(0, 25)) + String.fromCharCode(65 + randInt(0, 25));
    }

    combo = evaluateCombo(digits, letters);

    var baseRarity = kase.minRarity;
    var idx = RARITY_ORDER.indexOf(baseRarity);

    var upgradeChance = 0.02;
    if (state.adminLuckLevel > 0) upgradeChance += state.adminLuckLevel * 0.005;
    if (isEventActive('lucky')) upgradeChance += 0.03;
    if (isEventActive('mega')) upgradeChance += 0.08;
    if (upgradeChance > 0.15) upgradeChance = 0.10 + Math.random() * 0.05;

    for (var u = 0; u < 3; u++) {
      if (Math.random() < upgradeChance && idx < 4) {
        idx++;
        upgradeChance *= 0.4;
      }
    }
    finalRarity = RARITY_ORDER[Math.min(idx, 4)];
    price = computeBasePrice(finalRarity, region, false);

    // Супер-дроп
    var superRoll = Math.random();
    var superChance = 0.001;
    if (isEventActive('mega')) superChance += 0.005;
    if (state.adminLuckLevel > 0) superChance += state.adminLuckLevel * 0.0005;

    if (superRoll < superChance) {
      var tier = Math.random();
      if (tier < 0.7) price = RARITY_THRESHOLDS.rarest + Math.random() * (RARITY_THRESHOLDS.incredible - RARITY_THRESHOLDS.rarest);
      else if (tier < 0.95) price = RARITY_THRESHOLDS.incredible + Math.random() * (RARITY_THRESHOLDS.absolute - RARITY_THRESHOLDS.incredible);
      else price = RARITY_THRESHOLDS.absolute + Math.random() * 3000000000;
    }

    // БОГ
    var godChance = 0.00001;
    if (state.adminLuckLevel === 10) godChance += 0.00005;
    if (Math.random() < godChance) price = RARITY_THRESHOLDS.god;

    // Красивая комбинация
    if (price >= BEAUTIFUL_THRESHOLD) {
      var beautiful = generateBeautifulPlate(region);
      digits = beautiful.digits;
      letters = beautiful.letters;
      combo = evaluateCombo(digits, letters);
    }

    price *= state.adminCashMult;
    if (price >= RARITY_THRESHOLDS.god) price = RARITY_THRESHOLDS.god;
    price = Math.min(MAX_PRICE, Math.round(price));
    finalRarity = rarityFromPrice(price);

    if (isEventActive('candy') && Math.random() < 0.03) {
      isCandy = true;
      price = Math.round(price * 5);
      price = Math.min(MAX_PRICE, price);
      finalRarity = 'candy';
    }
  }

  var finalTags = combo.tags;
  if (isCandy) finalTags = '🍬 Конфетный номер · ' + finalTags;
  if (price >= BEAUTIFUL_THRESHOLD && !isManual) finalTags = '✨ КРАСИВЫЙ · ' + finalTags;

  state.current = {
    digits: digits, letters: letters,
    tags: finalTags, rarity: finalRarity, mult: combo.mult,
    value: price, region: region, isCandy: isCandy
  };
  state.rolled++;

  if (price >= SPECIAL_REVEAL_THRESHOLD) {
    showRevealScreen();
  } else {
    renderCurrent();
    playAppearAnimation(finalRarity);
    setSellEnabled(true);
  }

  save();
  updateUI();
}

// ===================== СПЕЦ-ЭКРАН =====================
var revealScreen = document.getElementById('revealScreen');
var revealBgGlow = document.getElementById('revealBgGlow');
var revealRarityMarks = document.getElementById('revealRarityMarks');
var revealRarityFill = document.getElementById('revealRarityFill');
var revealRarityText = document.getElementById('revealRarityText');
var revealPlate = document.getElementById('revealPlate');
var revealPlateMain = document.getElementById('revealPlateMain');
var revealPlateCode = document.getElementById('revealPlateCode');
var revealPlateCountry = document.getElementById('revealPlateCountry');
var revealPlateFlag = document.getElementById('revealPlateFlag');
var revealPriceWrap = document.getElementById('revealPriceWrap');
var revealPriceValue = document.getElementById('revealPriceValue');
var revealPriceTags = document.getElementById('revealPriceTags');
var revealActions = document.getElementById('revealActions');
var revealSellAmount = document.getElementById('revealSellAmount');
var revealTimers = [];

(function drawMarks() {
  var marks = [15, 30, 45, 60, 75, 85, 93, 98];
  for (var i = 0; i < marks.length; i++) {
    var m = document.createElement('div');
    m.className = 'reveal-rarity-mark';
    m.style.left = marks[i] + '%';
    revealRarityMarks.appendChild(m);
  }
})();

function rarityToProgress(rarity) {
  switch (rarity) {
    case 'common':     return 12;
    case 'rare':       return 28;
    case 'epic':       return 45;
    case 'mythic':     return 62;
    case 'legendary':  return 78;
    case 'rarest':     return 88;
    case 'incredible': return 94;
    case 'absolute':   return 98;
    case 'god':        return 100;
    case 'candy':      return 68;
    default:           return 10;
  }
}

// Заполнение шкалы — быстро, 2-3 секунды, с лёгким нагнетанием
function fillRarityBar(targetPercent, onDone) {
  var currentPercent = 0;
  var steps = 0;
  // Сколько "шагов" до конца
  var totalSteps = 30 + Math.floor(targetPercent / 6);

  function step() {
    if (currentPercent >= targetPercent) {
      revealRarityFill.style.width = targetPercent + '%';
      if (onDone) onDone();
      return;
    }

    var progress = steps / totalSteps;
    // Скорость: сначала быстро, потом замедление
    var speed = 5 * (1 - progress * 0.6) + Math.random() * 1.5;

    // Небольшая задержка иногда — нагнетание
    var delay = 40 + Math.random() * 40;
    if (Math.random() < 0.1 && progress > 0.5) {
      delay += 80 + Math.random() * 100;
    }

    currentPercent = Math.min(targetPercent, currentPercent + speed);
    revealRarityFill.style.width = currentPercent + '%';
    steps++;

    revealTimers.push(setTimeout(step, delay));
  }

  step();
}

function showRevealScreen() {
  var c = state.current;
  if (!c) return;

  revealScreen.classList.add('show');
  revealBgGlow.className = 'reveal-bg-glow';
  revealRarityFill.style.width = '0%';
  revealRarityText.className = 'reveal-rarity-text';
  revealRarityText.textContent = 'Сканирование...';
  revealPriceWrap.classList.remove('show');
  revealActions.classList.remove('show');
  revealPriceValue.className = 'reveal-price-value';
  revealPriceValue.textContent = '—';
  revealPriceTags.innerHTML = '';
  revealPlate.className = 'plate reveal-plate';

  var r = c.region;
  if (r.type === 'ru') {
    revealPlateMain.innerHTML = '<span class="l">' + c.letters[0] + '</span><span class="gap"></span><span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters.slice(1) + '</span>';
    revealPlateCountry.textContent = 'RUS'; revealPlateCode.textContent = r.regionCode; revealPlateFlag.textContent = '🇷🇺';
  } else if (r.type === 'by') {
    revealPlateMain.innerHTML = '<span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters + '</span>';
    revealPlateCountry.textContent = 'BY'; revealPlateCode.textContent = r.regionCode; revealPlateFlag.textContent = '🇧🇾';
  } else if (r.type === 'kz') {
    revealPlateMain.innerHTML = '<span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters + '</span>';
    revealPlateCountry.textContent = 'KZ'; revealPlateCode.textContent = r.regionCode; revealPlateFlag.textContent = '🇰🇿';
  } else if (r.type === 'ua') {
    revealPlateMain.innerHTML = '<span class="l">' + c.letters + '</span><span class="gap"></span><span class="d">' + c.digits + '</span>';
    revealPlateCountry.textContent = 'UA'; revealPlateCode.textContent = r.regionCode; revealPlateFlag.textContent = '🇺🇦';
  } else {
    revealPlateMain.innerHTML = '<span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters + '</span>';
    revealPlateCountry.textContent = r.countryCode; revealPlateCode.textContent = r.regionCode; revealPlateFlag.textContent = r.flag;
  }

  revealPlate.className = 'plate reveal-plate ' + (c.isCandy ? 'candy' : c.rarity);

  for (var i = 0; i < revealTimers.length; i++) clearTimeout(revealTimers[i]);
  revealTimers = [];

  var targetProgress = rarityToProgress(c.rarity);
  var isGod = c.rarity === 'god';

  if (targetProgress < 30) revealBgGlow.classList.add('tension-1');
  else if (targetProgress < 70) revealBgGlow.classList.add('tension-2');
  else revealBgGlow.classList.add('tension-3');

  // Заполняем шкалу ~2-3 секунды
  revealTimers.push(setTimeout(function() {
    fillRarityBar(targetProgress, function() {
      // Показываем редкость ПОСЛЕ заполнения
      var labels = {
        common: 'Обычный номер',
        rare: '💎 Редкий номер',
        epic: '💜 Эпический номер',
        mythic: '🔥 Мифический номер',
        legendary: '👑 Легендарный номер',
        rarest: '🌟 РЕДЧАЙШИЙ номер',
        incredible: '💠 НЕВЕРОЯТНЫЙ номер',
        absolute: '⚪ АБСОЛЮТНЫЙ номер',
        god: '⚡ БОГ ⚡',
        candy: '🍬 Конфетный номер'
      };
      revealRarityText.textContent = labels[c.rarity] || 'Номер';
      revealRarityText.classList.add('done');
      if (['rarest','incredible','absolute','god'].indexOf(c.rarity) !== -1) {
        revealRarityText.classList.add(c.rarity);
      }

      if (isGod) {
        revealPlate.className = 'plate reveal-plate ' + c.rarity;
        revealPlate.style.animation = 'revealPlateIn 1s cubic-bezier(.2,.9,.3,1.2), godShake 2.5s ease-out 0.5s infinite';
      }

      // ЦЕНА — строго ПОСЛЕ появления редкости, +1 секунда
      revealTimers.push(setTimeout(function() {
        revealPriceValue.textContent = formatMoney(c.value);
        if (isGod) revealPriceValue.classList.add('god');
        revealPriceTags.innerHTML = c.tags.split(' · ').map(function(tag) {
          return '<span class="tag">' + escapeHtml(tag) + '</span>';
        }).join('');
        revealPriceWrap.classList.add('show');

        // Кнопки ещё через 0.8 сек
        revealTimers.push(setTimeout(function() {
          revealSellAmount.textContent = formatMoney(c.value);
          revealActions.classList.add('show');
        }, 800));
      }, 900));
    });
  }, 300));
}

function hideRevealScreen() {
  for (var i = 0; i < revealTimers.length; i++) clearTimeout(revealTimers[i]);
  revealTimers = [];
  revealScreen.classList.remove('show');
}

document.getElementById('revealSell').onclick = function() {
  hideRevealScreen();
  sellNumber();
};

document.getElementById('revealSkip').onclick = function() {
  hideRevealScreen();
  renderCurrent();
  setSellEnabled(true);
  updateUI();
};

// ===================== ОТРИСОВКА =====================
function renderCurrent() {
  var c = state.current;
  if (!c) return;
  var r = c.region;
  var plateEl = document.getElementById('plate');
  var mainEl = document.getElementById('plateMain');
  var codeEl = document.getElementById('plateCode');
  var countryEl = document.getElementById('plateCountry');
  var flagEl = document.getElementById('plateFlag');
  var infoEl = document.getElementById('plateInfo');
  var valueEl = document.getElementById('plateValue');

  if (r.type === 'ru') {
    mainEl.innerHTML = '<span class="l">' + c.letters[0] + '</span><span class="gap"></span><span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters.slice(1) + '</span>';
    countryEl.textContent = 'RUS'; codeEl.textContent = r.regionCode; flagEl.textContent = '🇷🇺';
  } else if (r.type === 'by') {
    mainEl.innerHTML = '<span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters + '</span>';
    countryEl.textContent = 'BY'; codeEl.textContent = r.regionCode; flagEl.textContent = '🇧🇾';
  } else if (r.type === 'kz') {
    mainEl.innerHTML = '<span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters + '</span>';
    countryEl.textContent = 'KZ'; codeEl.textContent = r.regionCode; flagEl.textContent = '🇰🇿';
  } else if (r.type === 'ua') {
    mainEl.innerHTML = '<span class="l">' + c.letters + '</span><span class="gap"></span><span class="d">' + c.digits + '</span>';
    countryEl.textContent = 'UA'; codeEl.textContent = r.regionCode; flagEl.textContent = '🇺🇦';
  } else {
    mainEl.innerHTML = '<span class="d">' + c.digits + '</span><span class="gap"></span><span class="l">' + c.letters + '</span>';
    countryEl.textContent = r.countryCode; codeEl.textContent = r.regionCode; flagEl.textContent = r.flag;
  }

  var plateClass = 'plate ' + (c.isCandy ? 'candy' : c.rarity);
  if (c.value >= EXPENSIVE_THRESHOLD) plateClass += ' expensive';
  plateEl.className = plateClass;
  infoEl.textContent = c.tags;

  var extra = '';
  if (state.adminCashMult > 1) extra += ' <span style="color:#f5d76e">💰 ×' + state.adminCashMult + '</span>';
  if (c.value >= EXPENSIVE_THRESHOLD) extra += ' <span style="color:#ffd700;font-weight:900">⚠️</span>';

  valueEl.className = 'plate-value' + (c.value >= EXPENSIVE_THRESHOLD ? ' expensive-value' : '');
  valueEl.innerHTML = 'Оценка: <b style="color:' + (c.value >= EXPENSIVE_THRESHOLD ? '#ffd700' : '#6ee7a8') + '">' + formatMoney(c.value) + '</b> ' +
    '<span class="mult">(x' + c.mult.toFixed(1) + ')</span> ' +
    '<span class="rarity-tag ' + c.rarity + '">' + RARITY_LABELS[c.rarity] + '</span>' + extra;
}

function formatPlate(c) {
  var r = c.region;
  if (r.type === 'ru') return c.letters[0] + ' ' + c.digits + ' ' + c.letters.slice(1) + ' ' + r.regionCode;
  if (r.type === 'by') return c.digits + ' ' + c.letters + ' ' + r.regionCode;
  if (r.type === 'kz') return c.digits + ' ' + c.letters + ' ' + r.regionCode;
  if (r.type === 'ua') return c.letters + ' ' + c.digits + ' ' + r.regionCode;
  return c.digits + ' ' + c.letters + ' ' + r.regionCode;
}

function playAppearAnimation(rarity) {
  var plateEl = document.getElementById('plate');
  var infoEl = document.getElementById('plateInfo');
  var valueEl = document.getElementById('plateValue');
  var flashEl = document.getElementById('flash');

  plateEl.classList.remove('plate-animate','plate-rare-anim','plate-epic-anim','plate-mythic-anim','plate-legendary-anim','plate-rarest-anim','plate-incredible-anim','plate-absolute-anim','plate-god-anim','plate-candy-anim');
  infoEl.classList.remove('info-animate');
  valueEl.classList.remove('value-animate');
  flashEl.classList.remove('active','common','rare','epic','mythic','legendary','rarest','incredible','absolute','god','candy');

  void plateEl.offsetWidth;

  var animClass = 'plate-animate';
  if (rarity === 'rare') animClass = 'plate-rare-anim';
  else if (rarity === 'epic') animClass = 'plate-epic-anim';
  else if (rarity === 'mythic') animClass = 'plate-mythic-anim';
  else if (rarity === 'legendary') animClass = 'plate-legendary-anim';
  else if (rarity === 'rarest') animClass = 'plate-rarest-anim';
  else if (rarity === 'incredible') animClass = 'plate-incredible-anim';
  else if (rarity === 'absolute') animClass = 'plate-absolute-anim';
  else if (rarity === 'god') animClass = 'plate-god-anim';
  else if (rarity === 'candy') animClass = 'plate-candy-anim';
  plateEl.classList.add(animClass);
  infoEl.classList.add('info-animate');
  valueEl.classList.add('value-animate');

  if (rarity !== 'common') {
    flashEl.classList.add(rarity, 'active');
    setTimeout(function() { flashEl.classList.remove('active'); }, 700);
  }
}

function sellNumber() {
  if (!state.current) return;
  var c = state.current;
  state.balance += c.value;
  state.sold++;
  if (c.value > state.best) state.best = c.value;

  var cls = c.isCandy ? 'pink' : ({ common:'', rare:'blue', epic:'purple', mythic:'orange', legendary:'gold', rarest:'gold', incredible:'cyan', absolute:'white', god:'magenta' })[c.rarity];
  addLog('Продано: <b>' + c.region.flag + ' ' + formatPlate(c) + '</b> за <span class="' + cls + '">' + formatMoney(c.value) + '</span>' + (c.isCandy ? ' 🍬' : ''));

  state.current = null;
  document.getElementById('plateMain').innerHTML = '<span class="l">А</span><span class="gap"></span><span class="d">000</span><span class="gap"></span><span class="l">АА</span>';
  document.getElementById('plate').className = 'plate';
  document.getElementById('plateInfo').textContent = 'Выбери кейс и нажми «Сгенерировать»';
  document.getElementById('plateValue').className = 'plate-value';
  document.getElementById('plateValue').innerHTML = '';
  setSellEnabled(false);
  save();
  updateUI();
}

function updateUI() {
  document.getElementById('balance').textContent = formatMoney(state.balance);
  document.getElementById('sold').textContent = state.sold;
  document.getElementById('best').textContent = state.best ? formatMoney(state.best) : '—';
  document.getElementById('rolled').textContent = state.rolled;

  var kase = null;
  for (var i = 0; i < CASES.length; i++) if (CASES[i].id === state.selectedCase) kase = CASES[i];
  if (!kase) kase = CASES[0];

  var btnGen = document.getElementById('btnGen');
  if (state.balance < kase.price) {
    btnGen.disabled = true;
    btnGen.textContent = '🎲 Нужно ' + formatMoney(kase.price);
  } else {
    btnGen.disabled = false;
    btnGen.textContent = '🎲 Сгенерировать (' + formatMoney(kase.price) + ')';
  }

  document.querySelectorAll('.case-item').forEach(function(el) {
    var id = el.dataset.caseId;
    var c = null;
    for (var j = 0; j < CASES.length; j++) if (CASES[j].id === id) c = CASES[j];
    if (!c) return;
    el.classList.toggle('active', id === state.selectedCase);
    if (state.balance < c.price) el.classList.add('disabled');
    else el.classList.remove('disabled');
  });
}

function setSellEnabled(enabled) {
  document.getElementById('btnSell').disabled = !enabled;
}

function addLog(text) {
  var log = document.getElementById('log');
  var div = document.createElement('div');
  div.className = 'log-item';
  div.innerHTML = text;
  log.prepend(div);
  while (log.children.length > 30) log.removeChild(log.lastChild);
}

function renderRegions() {
  var container = document.getElementById('regions');
  container.innerHTML = '';
  REGIONS.forEach(function(r) {
    var btn = document.createElement('button');
    btn.className = 'region-btn' + (r.id === state.region.id ? ' active' : '');
    btn.innerHTML = '<span class="flag">' + r.flag + '</span>' + r.name;
    btn.onclick = function() {
      state.region = r;
      state.current = null;
      document.getElementById('plateMain').innerHTML = '<span class="l">А</span><span class="gap"></span><span class="d">000</span><span class="gap"></span><span class="l">АА</span>';
      document.getElementById('plate').className = 'plate';
      document.getElementById('plateInfo').textContent = 'Выбери кейс и нажми «Сгенерировать»';
      document.getElementById('plateValue').className = 'plate-value';
      document.getElementById('plateValue').innerHTML = '';
      setSellEnabled(false);
      renderRegions();
      addLog('Регион изменён: <b>' + r.flag + ' ' + r.name + '</b>');
      save();
    };
    container.appendChild(btn);
  });
}

function renderCases() {
  var grid = document.getElementById('caseGrid');
  grid.innerHTML = '';
  CASES.forEach(function(c) {
    var el = document.createElement('div');
    el.className = 'case-item c-' + c.minRarity;
    if (state.balance < c.price) el.classList.add('disabled');
    if (state.selectedCase === c.id) el.classList.add('active');
    el.dataset.caseId = c.id;
    var locked = state.balance < c.price ? '<div class="case-locked">не хватает</div>' : '';
    el.innerHTML = '<div class="case-icon">' + c.icon + '</div>' +
                   '<div class="case-name">' + c.name + '</div>' +
                   '<div class="case-price">' + formatMoney(c.price) + '</div>' + locked;
    el.onclick = function() {
      if (el.classList.contains('disabled')) return;
      state.selectedCase = c.id;
      save();
      renderCases();
      updateUI();
    };
    grid.appendChild(el);
  });
}

var confirmOverlay = document.getElementById('confirmOverlay');
var confirmCallback = null;

function showConfirm(callback) {
  var c = state.current;
  if (!c) { callback(); return; }
  document.getElementById('confirmPlate').textContent = formatPlate(c);
  document.getElementById('confirmValue').textContent = formatMoney(c.value);
  confirmCallback = callback;
  confirmOverlay.classList.add('show');
}

document.getElementById('confirmCancel').onclick = function() {
  confirmOverlay.classList.remove('show');
  confirmCallback = null;
  addLog('⚠️ Прокрутка отменена. Продай дорогой номер!');
};

document.getElementById('confirmGo').onclick = function() {
  confirmOverlay.classList.remove('show');
  var cb = confirmCallback;
  confirmCallback = null;
  if (cb) cb();
};

function handleGenerateClick() {
  if (state.current && state.current.value >= EXPENSIVE_THRESHOLD) {
    showConfirm(function() {
      state.current = null;
      openCase(state.selectedCase);
    });
  } else {
    state.current = null;
    openCase(state.selectedCase);
  }
}

// ===================== АДМИН =====================
var adminPassOverlay = document.getElementById('adminPassOverlay');
var adminPassInput = document.getElementById('adminPassInput');
var adminPassError = document.getElementById('adminPassError');
var adminOverlay = document.getElementById('adminOverlay');
var isAdminUnlocked = false;

function requestAdminPassword() {
  if (isAdminUnlocked) {
    adminOverlay.classList.add('show');
    updateAdminUI();
    return;
  }
  adminPassInput.value = '';
  adminPassError.textContent = '';
  adminPassOverlay.classList.add('show');
  setTimeout(function() { adminPassInput.focus(); }, 100);
}

document.getElementById('adminPassCancel').onclick = function() {
  adminPassOverlay.classList.remove('show');
  adminPassInput.value = '';
  adminPassError.textContent = '';
};

document.getElementById('adminPassOk').onclick = function() {
  var val = adminPassInput.value;
  if (val === ADMIN_PASSWORD) {
    isAdminUnlocked = true;
    adminPassOverlay.classList.remove('show');
    adminOverlay.classList.add('show');
    updateAdminUI();
    addLog('🔓 Админ-панель разблокирована');
  } else {
    adminPassError.textContent = 'Неверный пароль';
    adminPassInput.value = '';
    adminPassInput.focus();
  }
};

adminPassInput.addEventListener('keydown', function(e) {
  if (e.key === 'Enter') document.getElementById('adminPassOk').click();
});

document.addEventListener('keydown', function(e) {
  if (e.ctrlKey && e.shiftKey && (e.key === 'A' || e.key === 'a' || e.code === 'KeyA')) {
    e.preventDefault();
    if (adminOverlay.classList.contains('show')) adminOverlay.classList.remove('show');
    else requestAdminPassword();
  }
});

var titleTaps = 0, titleTimer = null;
document.getElementById('gameTitle').addEventListener('click', function() {
  titleTaps++;
  clearTimeout(titleTimer);
  titleTimer = setTimeout(function() { titleTaps = 0; }, 800);
  if (titleTaps >= 5) { titleTaps = 0; requestAdminPassword(); }
});

document.getElementById('adminClose').onclick = function() { adminOverlay.classList.remove('show'); };
adminOverlay.addEventListener('click', function(e) { if (e.target === adminOverlay) adminOverlay.classList.remove('show'); });

// Ручной номер
var manualRaritySel = document.getElementById('manualRarity');
var manualPriceInp = document.getElementById('manualPrice');
var manualBadge = document.getElementById('manualBadge');

document.getElementById('manualSetBtn').onclick = function() {
  if (!isAdminUnlocked) return;
  var rar = manualRaritySel.value || null;
  var prc = manualPriceInp.value ? Math.max(100, Math.min(MAX_PRICE, Math.floor(+manualPriceInp.value))) : null;
  if (!rar && !prc) {
    addLog('⚠️ Укажите редкость или цену');
    return;
  }
  state.manualRarity = rar;
  state.manualPrice = prc;
  updateManualBadge();
  addLog('⚙️ Следующий номер: редкость <b>' + (rar ? RARITY_LABELS[rar] : '—') + '</b>, цена <b>' + (prc ? formatMoney(prc) : 'по редкости') + '</b>');
};

document.getElementById('manualClearBtn').onclick = function() {
  if (!isAdminUnlocked) return;
  state.manualRarity = null;
  state.manualPrice = null;
  manualRaritySel.value = '';
  manualPriceInp.value = '';
  updateManualBadge();
  addLog('⚙️ Ручная настройка сброшена');
};

function updateManualBadge() {
  if (state.manualRarity || state.manualPrice) manualBadge.classList.add('active');
  else manualBadge.classList.remove('active');
}

// Удача
var luckSlider = document.getElementById('luckLevelSlider');
var luckValEl = document.getElementById('luckLevelVal');
luckSlider.addEventListener('input', function() {
  if (!isAdminUnlocked) return;
  state.adminLuckLevel = +luckSlider.value;
  luckValEl.textContent = state.adminLuckLevel + ' / 10';
  save(); renderEventsBanner();
});

document.querySelectorAll('.mult-btn').forEach(function(btn) {
  btn.onclick = function() {
    if (!isAdminUnlocked) return;
    state.adminCashMult = +btn.dataset.mult;
    save(); updateAdminUI(); renderEventsBanner();
    addLog('💰 Множитель кэша: <b>x' + state.adminCashMult + '</b>');
  };
});

function updateAdminUI() {
  luckSlider.value = state.adminLuckLevel;
  luckValEl.textContent = state.adminLuckLevel + ' / 10';
  document.querySelectorAll('.mult-btn').forEach(function(btn) {
    btn.classList.toggle('active', +btn.dataset.mult === state.adminCashMult);
  });
  updateManualBadge();
}

var broadcastEl = document.getElementById('adminBroadcast');
var broadcastTextEl = document.getElementById('adminBroadcastText');
var broadcastHideTimer = null;

function showBroadcast(text, color, durationSec) {
  if (!text || !text.trim()) return;
  broadcastTextEl.textContent = text.trim();
  broadcastEl.className = 'admin-broadcast color-' + (color || 'purple');
  void broadcastEl.offsetWidth;
  broadcastEl.classList.add('show');
  if (broadcastHideTimer) clearTimeout(broadcastHideTimer);
  if (durationSec > 0) {
    broadcastHideTimer = setTimeout(function() { broadcastEl.classList.remove('show'); }, durationSec * 1000);
  }
}

document.getElementById('broadcastBtn').onclick = function() {
  if (!isAdminUnlocked) return;
  var text = document.getElementById('broadcastText').value;
  var color = document.getElementById('broadcastColor').value;
  var dur = +document.getElementById('broadcastDur').value;
  if (!text.trim()) return;
  showBroadcast(text, color, dur);
  addLog('📢 <b>АДМИН:</b> ' + escapeHtml(text));
};

broadcastEl.addEventListener('click', function() {
  if (broadcastHideTimer) clearTimeout(broadcastHideTimer);
  broadcastEl.classList.remove('show');
});

document.querySelectorAll('.admin-btn.start').forEach(function(btn) {
  btn.onclick = function() {
    if (!isAdminUnlocked) return;
    var name = btn.dataset.event;
    var durSel = document.getElementById(name + 'Dur');
    var dur = +durSel.value;
    state.events[name] = Date.now() + dur * 1000;
    save(); renderEventsBanner();
    addLog('🎉 Запущен ивент: <b>' + EVENT_INFO[name].label + '</b> на ' + dur + ' сек.');
  };
});
document.querySelectorAll('.admin-btn.stop').forEach(function(btn) {
  btn.onclick = function() {
    if (!isAdminUnlocked) return;
    var name = btn.dataset.event;
    delete state.events[name];
    save(); renderEventsBanner();
    addLog('⛔ Ивент <b>' + EVENT_INFO[name].label + '</b> остановлен.');
  };
});

document.getElementById('giveBtn').onclick = function() {
  if (!isAdminUnlocked) return;
  var amount = Math.max(0, Math.floor(+document.getElementById('giveAmount').value || 0));
  if (!amount) return;
  state.balance += amount;
  addLog('🛠 Админ выдал <b>' + formatMoney(amount) + '</b>');
  save(); updateUI();
};

document.getElementById('clearLogBtn').onclick = function() {
  if (!isAdminUnlocked) return;
  document.getElementById('log').innerHTML = '';
  addLog('🧹 Лог очищен');
};

document.getElementById('btnGen').onclick = handleGenerateClick;
document.getElementById('btnSell').onclick = sellNumber;

document.getElementById('resetBtn').onclick = function() {
  if (!confirm('Точно сбросить весь прогресс?')) return;
  state.balance = START_BALANCE;
  state.sold = 0; state.best = 0; state.rolled = 0;
  state.region = REGIONS[0];
  state.current = null;
  state.selectedCase = 'common';
  state.events = {};
  state.adminLuckLevel = 0;
  state.adminCashMult = 1;
  state.manualRarity = null;
  state.manualPrice = null;
  Store.removeItem(SAVE_KEY);
  document.getElementById('plateMain').innerHTML = '<span class="l">А</span><span class="gap"></span><span class="d">000</span><span class="gap"></span><span class="l">АА</span>';
  document.getElementById('plate').className = 'plate';
  document.getElementById('plateInfo').textContent = 'Выбери кейс и нажми «Сгенерировать»';
  document.getElementById('plateValue').className = 'plate-value';
  document.getElementById('plateValue').innerHTML = '';
  document.getElementById('log').innerHTML = '<div class="log-item">Прогресс сброшен. Выдан стартовый капитал 100 000 ₽!</div>';
  setSellEnabled(false);
  renderRegions(); renderCases(); renderEventsBanner();
  updateAdminUI(); updateUI();
  save();
};

setInterval(function() {
  var now = Date.now();
  var changed = false;
  for (var k in state.events) {
    if (state.events[k] <= now) { delete state.events[k]; changed = true; }
  }
  if (changed) save();
  renderEventsBanner();
}, 1000);

var hadProgress = load();
var welcomeOverlay = document.getElementById('welcomeOverlay');
var gameScreen = document.getElementById('gameScreen');

if (hadProgress) {
  welcomeOverlay.classList.add('hide');
  gameScreen.style.display = 'block';
} else {
  document.getElementById('welcomeStart').onclick = function() {
    welcomeOverlay.classList.add('hide');
    gameScreen.style.display = 'block';
  };
}

renderRegions(); renderCases(); renderEventsBanner();
updateAdminUI(); updateUI();

if (hadProgress) {
  addLog('💾 Прогресс восстановлен: ' + formatMoney(state.balance));
} else {
  addLog('🎁 Выдан стартовый капитал ' + formatMoney(START_BALANCE) + '. Начинай крутить!');
}

})();
</script>
</body>
</html>
