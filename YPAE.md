import React, { useState, useEffect, useRef } from 'react';
import { 
  Rocket, 
  Info, 
  RefreshCw, 
  Play, 
  Pause, 
  HelpCircle, 
  Flame, 
  Globe, 
  Gauge, 
  Compass, 
  TrendingUp, 
  Award,
  ChevronRight,
  Sparkles,
  ShieldAlert
} from 'lucide-react';

// --- DATA & PHYSICS CONSTANTS ---
const PLANETS = {
  earth: {
    name: 'Earth',
    gravity: 9.81, // m/s^2
    atmosphereDensity: 1.225, // kg/m^3
    scaleHeight: 8500, // meters
    escapeVelocity: 11186, // m/s
    radius: 6371000, // meters
    color: '#3B82F6',
    bgColor: 'bg-blue-900',
    skyColor: 'rgba(59, 130, 246, 0.4)',
    groundColor: '#10B981',
    description: 'Perfect moderate gravity with a standard atmosphere. The benchmark for rocket design.',
    fact: 'Requires a solid TWR > 1.2 at launch to counter gravity and atmospheric drag.'
  },
  mars: {
    name: 'Mars',
    gravity: 3.71,
    atmosphereDensity: 0.02, // extremely thin
    scaleHeight: 11100,
    escapeVelocity: 5027,
    radius: 3389500,
    color: '#EF4444',
    bgColor: 'bg-red-900',
    skyColor: 'rgba(239, 68, 68, 0.2)',
    groundColor: '#92400E',
    description: 'Lower gravity and exceptionally thin atmosphere makes takeoffs significantly easier.',
    fact: 'Aerodynamic drag is almost negligible here, allowing highly efficient, low-thrust takeoffs.'
  },
  moon: {
    name: 'The Moon',
    gravity: 1.62,
    atmosphereDensity: 0.0, // perfect vacuum
    scaleHeight: 1, // irrelevant but avoids div by zero
    escapeVelocity: 2380,
    radius: 1737400,
    color: '#9CA3AF',
    bgColor: 'bg-gray-800',
    skyColor: 'rgba(0, 0, 0, 0)',
    groundColor: '#4B5563',
    description: 'Low gravity with absolutely no atmosphere. Ideal playground for vacuum-optimized rockets.',
    fact: 'No atmospheric drag whatsoever. Escape velocity can be achieved with very little fuel.'
  },
  venus: {
    name: 'Venus',
    gravity: 8.87,
    atmosphereDensity: 65.0, // crushing density!
    scaleHeight: 15900,
    escapeVelocity: 10360,
    radius: 6051800,
    color: '#F59E0B',
    bgColor: 'bg-amber-950',
    skyColor: 'rgba(245, 158, 11, 0.5)',
    groundColor: '#78350F',
    description: 'Crushing atmospheric density (65x Earth). Drag will violently slow your ascent.',
    fact: 'Massive atmospheric drag creates terminal limits. High aerodynamic profiles suffer heavily.'
  },
  mercury: {
    name: 'Mercury',
    gravity: 3.7,
    atmosphereDensity: 0.0,
    scaleHeight: 1,
    escapeVelocity: 4250,
    radius: 2439700,
    color: '#6B7280',
    bgColor: 'bg-stone-900',
    skyColor: 'rgba(20, 20, 20, 0)',
    groundColor: '#374151',
    description: 'Low gravity environment with no atmosphere, baking right next to the Sun.',
    fact: 'Similar to the Moon, but with more gravity. Excellent place for high efficiency low-thrust systems.'
  },
  jupiter: {
    name: 'Jupiter (1-Bar Level)',
    gravity: 24.79, // massive!
    atmosphereDensity: 0.16, // at 1 bar level, but high gravity
    scaleHeight: 27000,
    escapeVelocity: 59500,
    radius: 69911000,
    color: '#D97706',
    bgColor: 'bg-amber-900',
    skyColor: 'rgba(217, 119, 6, 0.35)',
    groundColor: '#92400E',
    description: 'Immense gravity with no solid surface. Escaping requires colossal thrust.',
    fact: 'Standard rockets cannot take off from here. You need extreme engines to exceed its massive 24.79 m/s².'
  },
  saturn: {
    name: 'Saturn (1-Bar Level)',
    gravity: 10.44,
    atmosphereDensity: 0.19,
    scaleHeight: 59500,
    escapeVelocity: 35500,
    radius: 58232000,
    color: '#FBBF24',
    bgColor: 'bg-yellow-950',
    skyColor: 'rgba(251, 191, 36, 0.25)',
    groundColor: '#78350F',
    description: 'Gravity similar to Earth, but enormous size makes escape velocity extremely high.',
    fact: 'Takes off easily at first due to moderate gravity, but reaching space orbit is a massive marathon.'
  },
  uranus: {
    name: 'Uranus',
    gravity: 8.69,
    atmosphereDensity: 0.42,
    scaleHeight: 27700,
    escapeVelocity: 21300,
    radius: 25362000,
    color: '#22D3EE',
    bgColor: 'bg-cyan-950',
    skyColor: 'rgba(34, 211, 238, 0.25)',
    groundColor: '#0891B2',
    description: 'Colder, icy gas giant with moderate gravity but dense cold hydrogen atmosphere.',
    fact: 'Moderate gravity but high scale height creates persistent low-level drag on ascent.'
  },
  neptune: {
    name: 'Neptune',
    gravity: 11.15,
    atmosphereDensity: 0.45,
    scaleHeight: 19700,
    escapeVelocity: 23500,
    radius: 24622000,
    color: '#3B82F6',
    bgColor: 'bg-indigo-950',
    skyColor: 'rgba(59, 130, 246, 0.25)',
    groundColor: '#1D4ED8',
    description: 'High wind speeds, dense atmosphere, and gravity stronger than Earth.',
    fact: 'High starting gravity and dense atmospheric gases make achieving orbit highly fuel-intensive.'
  }
};

const PRESETS = [
  {
    name: 'Small Sat Launcher',
    dryMass: 1500, // kg
    fuelMass: 12000, // kg
    thrust: 180, // kN
    isp: 310, // seconds
    description: 'Lightweight orbital launcher ideal for small payloads on moderate worlds.'
  },
  {
    name: 'Heavy Heavy Lifter',
    dryMass: 50000,
    fuelMass: 650000,
    thrust: 9500,
    isp: 380,
    description: 'A monster inspired by the Saturn V first stage, packing immense raw energy.'
  },
  {
    name: 'Martian Ascent Vehicle',
    dryMass: 2500,
    fuelMass: 8000,
    thrust: 60,
    isp: 340,
    description: 'Optimized for thin atmospheres and lower gravity landings & departures.'
  },
  {
    name: 'Fusion Powered Ark',
    dryMass: 150000,
    fuelMass: 300000,
    thrust: 15000,
    isp: 850,
    description: 'Sci-fi high-efficiency concept engine with supreme specific impulse (Isp).'
  }
];

const MISSIONS = [
  {
    id: 'm1',
    title: 'Earth Ascent Test',
    planet: 'earth',
    targetAltitude: 10000, // 10 km
    description: 'Launch your rocket and clear 10,000 meters altitude on Earth.',
    tip: 'Make sure your Thrust-to-Weight Ratio (TWR) is at least 1.2!'
  },
  {
    id: 'm2',
    title: 'Escape the Furnace',
    planet: 'venus',
    targetAltitude: 15000,
    description: 'Fight the extreme 65.0 kg/m³ atmospheric density of Venus and reach 15 km altitude.',
    tip: 'Venus requires brute-force thrust and clean aerodynamics. Lightweight dry masses fare better!'
  },
  {
    id: 'm3',
    title: 'Martian Orbit Insertion',
    planet: 'mars',
    targetAltitude: 25000,
    description: 'Ascend to 25,000 meters on Mars where the atmosphere is completely gone.',
    tip: 'Mars is forgiving! You can lift off with much lower engine thrust configurations.'
  },
  {
    id: 'm4',
    title: 'Conquer the Giant',
    planet: 'jupiter',
    targetAltitude: 5000,
    description: 'LIFTOFF from Jupiter and reach 5,000 meters high. The supreme challenge.',
    tip: 'Jupiter has 24.79 m/s² gravity. You need massive thrust and a very high fuel ratio to lift off at all!'
  }
];

export default function App() {
  // --- STATE ---
  const [selectedPlanetKey, setSelectedPlanetKey] = useState('earth');
  const [dryMass, setDryMass] = useState(15000); // kg
  const [fuelMass, setFuelMass] = useState(100000); // kg
  const [engineThrust, setEngineThrust] = useState(2500); // kN
  const [isp, setIsp] = useState(320); // seconds (vacuum/average specific impulse)
  const [dragCoef, setDragCoef] = useState(0.3); // Cd
  const [crossSection, setCrossSection] = useState(7.0); // m^2 (frontal area)

  // Simulation run states
  const [isRunning, setIsRunning] = useState(false);
  const [altitude, setAltitude] = useState(0); // m
  const [velocity, setVelocity] = useState(0); // m/s
  const [currentFuel, setCurrentFuel] = useState(100000); // kg
  const [flightTime, setFlightTime] = useState(0); // seconds
  const [maxAltitude, setMaxAltitude] = useState(0);
  const [maxVelocity, setMaxVelocity] = useState(0);
  const [flightStatus, setFlightStatus] = useState('grounded'); // grounded, launching, flying, coasting, crashed, escaped, outOfFuel
  const [crashReason, setCrashReason] = useState('');

  // UI state
  const [activeTab, setActiveTab] = useState('presets'); // presets, advanced
  const [selectedPreset, setSelectedPreset] = useState(-1);
  const [activeMissionId, setActiveMissionId] = useState('m1');
  
  // Real-time calculated values
  const planet = PLANETS[selectedPlanetKey];
  const totalMass = dryMass + currentFuel;
  const gravityForce = totalMass * planet.gravity;
  const initialTotalMass = dryMass + fuelMass;
  
  // Instantaneous Calculations
  const currentTWR = (engineThrust * 1000) / (totalMass * planet.gravity);
  const initialTWR = (engineThrust * 1000) / (initialTotalMass * planet.gravity);

  // References for Animation & Canvas
  const canvasRef = useRef(null);
  const requestRef = useRef(null);
  const lastTimeRef = useRef(null);
  const particlesRef = useRef([]);

  // Load Preset
  const applyPreset = (preset, idx) => {
    setDryMass(preset.dryMass);
    setFuelMass(preset.fuelMass);
    setCurrentFuel(preset.fuelMass);
    setEngineThrust(preset.thrust);
    setIsp(preset.isp);
    setSelectedPreset(idx);
    resetSimulation();
  };

  // Reset Simulation
  const resetSimulation = () => {
    setIsRunning(false);
    setAltitude(0);
    setVelocity(0);
    setCurrentFuel(fuelMass);
    setFlightTime(0);
    setMaxAltitude(0);
    setMaxVelocity(0);
    setFlightStatus('grounded');
    setCrashReason('');
    particlesRef.current = [];
  };

  // Keep currentFuel aligned if fuelMass changes when grounded
  useEffect(() => {
    if (flightStatus === 'grounded') {
      setCurrentFuel(fuelMass);
    }
  }, [fuelMass, flightStatus]);

  // Load Mission Configuration
  const startMission = (mission) => {
    setActiveMissionId(mission.id);
    setSelectedPlanetKey(mission.planet);
    resetSimulation();
  };

  // --- PHYSICS ENGINE LOOP ---
  const updatePhysics = (deltaTime) => {
    // Cap deltaTime to avoid massive jumps on lag
    const dt = Math.min(deltaTime, 0.1); 

    setFlightTime(prev => prev + dt);

    setAltitude(prevAlt => {
      setCurrentFuel(prevFuel => {
        setVelocity(prevVel => {
          let fuel = prevFuel;
          let vel = prevVel;
          let alt = prevAlt;

          // 1. Calculate Fuel Burn & Thrust
          let thrustActive = isRunning && fuel > 0 && flightStatus !== 'crashed';
          let activeThrustForce = 0;
          let massFlowRate = 0;

          if (thrustActive) {
            // massFlowRate = Thrust / (Isp * g_earth)
            // Note: Specific impulse standard uses Earth gravity 9.80665 m/s² for exhaust speed relation
            massFlowRate = (engineThrust * 1000) / (isp * 9.80665);
            const fuelBurned = massFlowRate * dt;
            
            if (fuel >= fuelBurned) {
              fuel -= fuelBurned;
              activeThrustForce = engineThrust * 1000; // Newtons
            } else {
              // Burn the last of the fuel
              const fraction = fuel / fuelBurned;
              activeThrustForce = engineThrust * 1000 * fraction;
              fuel = 0;
            }
          }

          const currentTotalMass = dryMass + fuel;

          // 2. Gravitational Force (reduces slightly with altitude via inverse-square law)
          const localGravity = planet.gravity * Math.pow(planet.radius / (planet.radius + alt), 2);
          const weight = currentTotalMass * localGravity;

          // 3. Atmospheric Density & Aerodynamic Drag
          let density = 0;
          if (planet.atmosphereDensity > 0) {
            // Barometric formula: rho = rho_0 * e^(-altitude / scaleHeight)
            density = planet.atmosphereDensity * Math.exp(-alt / planet.scaleHeight);
          }
          const dragForce = 0.5 * density * Math.pow(vel, 2) * dragCoef * crossSection;
          // Drag opposes direction of velocity
          const dragDirection = vel > 0 ? -1 : 1;
          const netDragForce = dragForce * dragDirection;

          // 4. Net Acceleration
          // F_net = Thrust - Weight + Drag_force
          const netForce = activeThrustForce - weight + (vel !== 0 ? netDragForce : 0);
          const accel = netForce / currentTotalMass;

          // Apply Euler-Cromer integration
          let newVel = vel + accel * dt;

          // Prevent launching down below surface
          let newAlt = alt + newVel * dt;

          if (newAlt <= 0) {
            newAlt = 0;
            // Check for hard crash on landing
            if (newVel < -8) {
              setFlightStatus('crashed');
              setCrashReason(`Extreme hard impact! Rocket hit the ground at ${Math.abs(Math.round(newVel))} m/s.`);
              setIsRunning(false);
              newVel = 0;
            } else {
              newVel = 0;
              if (flightStatus !== 'grounded') {
                setFlightStatus('grounded');
                setIsRunning(false);
              }
            }
          } else {
            // Update flying statuses
            if (thrustActive) {
              setFlightStatus('launching');
            } else if (fuel <= 0 && newVel > 0) {
              setFlightStatus('outOfFuel');
            } else if (newVel <= 0) {
              setFlightStatus('coasting');
            } else {
              setFlightStatus('flying');
            }
          }

          // Check for Planet Escape Velocity!
          if (newVel >= planet.escapeVelocity) {
            setFlightStatus('escaped');
            setIsRunning(false);
          }

          // Record records
          if (newAlt > maxAltitude) setMaxAltitude(Math.round(newAlt));
          if (Math.abs(newVel) > maxVelocity) setMaxVelocity(Math.round(Math.abs(newVel)));

          // Exhaust particles generation
          if (thrustActive && Math.random() < 0.6) {
            const particleCount = Math.floor(activeThrustForce / 100000) + 2;
            for (let i = 0; i < Math.min(particleCount, 8); i++) {
              particlesRef.current.push({
                x: 0 + (Math.random() - 0.5) * 8,
                y: newAlt,
                vx: (Math.random() - 0.5) * 15,
                vy: -newVel - (Math.random() * 50 + 20),
                life: 1.0,
                size: Math.random() * 8 + 4
              });
            }
          }

          // Return values back to state via state batching hacks or letting ref handle temporary updates
          // Actually updating velocity here
          return newVel;
        });
        return fuel;
      });
      return newAlt;
    });
  };

  // Animation Loop Hook
  const loop = (time) => {
    if (lastTimeRef.current !== null) {
      const deltaTime = (time - lastTimeRef.current) / 1000;
      if (isRunning && flightStatus !== 'crashed' && flightStatus !== 'escaped') {
        updatePhysics(deltaTime);
      }
    }
    lastTimeRef.current = time;
    drawCanvas();
    requestRef.current = requestAnimationFrame(loop);
  };

  useEffect(() => {
    requestRef.current = requestAnimationFrame(loop);
    return () => cancelAnimationFrame(requestRef.current);
  }, [isRunning, flightStatus, selectedPlanetKey, dryMass, fuelMass, engineThrust, isp]);

  // --- CANVAS DRAWING ---
  const drawCanvas = () => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;

    // Clear Screen
    ctx.clearRect(0, 0, width, height);

    // Update & clean up particles
    particlesRef.current.forEach(p => {
      p.x += p.vx * 0.05;
      // Convert real altitude coordinate movement to pixel movement relative to screen
      p.y += p.vy * 0.05;
      p.life -= 0.02;
    });
    particlesRef.current = particlesRef.current.filter(p => p.life > 0);

    // 1. Draw Space Sky & Atmospheric Gradient
    // As altitude increases, atmosphere color blends to black space
    const atmosphereThickness = planet.scaleHeight * 4; // approximate visual height limit
    const atmFactor = Math.max(0, 1 - (altitude / atmosphereThickness));
    
    const skyGrad = ctx.createLinearGradient(0, 0, 0, height);
    if (planet.atmosphereDensity > 0) {
      // Dark space on top, atmospheric color on bottom depending on density factor
      skyGrad.addColorStop(0, '#030712'); // Pure dark space
      const atmOpacity = 0.9 * atmFactor;
      skyGrad.addColorStop(1, planet.atmosphereDensity > 10 
        ? `rgba(180, 83, 9, ${atmOpacity})` // Venus thick orange
        : `rgba(59, 130, 246, ${atmOpacity})` // Earth blue
      );
    } else {
      skyGrad.addColorStop(0, '#030712');
      skyGrad.addColorStop(1, '#111827');
    }
    ctx.fillStyle = skyGrad;
    ctx.fillRect(0, 0, width, height);

    // Draw Stars (only visible when atmosphere is thin)
    ctx.fillStyle = `rgba(255, 255, 255, ${1 - atmFactor})`;
    for (let i = 0; i < 40; i++) {
      const starX = (Math.sin(i * 4324) * 0.5 + 0.5) * width;
      const starY = (Math.cos(i * 9211) * 0.5 + 0.5) * height;
      ctx.fillRect(starX, starY, 1.5, 1.5);
    }

    // 2. Draw Ground (Parallax Scrolling)
    // The ground moves down out of view as rocket ascends
    const groundHeight = 80;
    const groundY = height - groundHeight + (altitude * 0.2); // ground recedes slower for depth
    
    if (groundY < height + 100) {
      // Ground fills bottom
      ctx.fillStyle = planet.groundColor;
      ctx.beginPath();
      ctx.rect(0, groundY, width, height - groundY + 100);
      ctx.fill();

      // Simple mountains / surface bumps on ground
      ctx.fillStyle = 'rgba(0, 0, 0, 0.15)';
      ctx.beginPath();
      ctx.moveTo(0, groundY);
      for (let x = 0; x <= width; x += 40) {
        const h = Math.sin(x * 0.05) * 15;
        ctx.lineTo(x, groundY - h);
      }
      ctx.lineTo(width, height);
      ctx.lineTo(0, height);
      ctx.closePath();
      ctx.fill();
    }

    // 3. Draw Flame & Smoke Particles
    particlesRef.current.forEach(p => {
      // Particle Y position relative to rocket altitude
      // Rocket is fixed at 1/3 from the bottom of screen (or locked to ground if altitude is low)
      const rocketScreenY = getRocketScreenY(height);
      const relativeY = rocketScreenY + (altitude - p.y) * 0.4; // scale altitude to pixel coordinate

      if (relativeY >= 0 && relativeY <= height) {
        ctx.fillStyle = `rgba(249, 115, 22, ${p.life})`;
        ctx.beginPath();
        ctx.arc(width / 2 + p.x, relativeY, p.size * p.life, 0, Math.PI * 2);
        ctx.fill();

        // Hot yellow core for flame
        if (p.life > 0.6) {
          ctx.fillStyle = `rgba(253, 224, 71, ${p.life})`;
          ctx.beginPath();
          ctx.arc(width / 2 + p.x * 0.5, relativeY, p.size * 0.5 * p.life, 0, Math.PI * 2);
          ctx.fill();
        }
      }
    });

    // 4. Draw Rocket
    const rocketY = getRocketScreenY(height);
    drawRocketSprite(ctx, width / 2, rocketY);

    // 5. Draw Target Goal Line for Active Mission
    const currentMission = MISSIONS.find(m => m.id === activeMissionId);
    if (currentMission && currentMission.planet === selectedPlanetKey) {
      const targetAlt = currentMission.targetAltitude;
      // Draw target indicator line if it fits within reasonable screen distance
      const targetY = rocketY - (targetAlt - altitude) * 0.4;
      if (targetY > 20 && targetY < height - 20) {
        ctx.strokeStyle = 'rgba(16, 185, 129, 0.6)';
        ctx.setLineDash([5, 5]);
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(10, targetY);
        ctx.lineTo(width - 10, targetY);
        ctx.stroke();
        ctx.setLineDash([]); // reset

        ctx.fillStyle = '#10B981';
        ctx.font = '11px sans-serif';
        ctx.fillText(`TARGET ALTITUDE: ${targetAlt}m`, 15, targetY - 6);
      }
    }
  };

  const getRocketScreenY = (height) => {
    // If on ground or very low, anchor rocket to ground level
    // Once higher, anchor rocket to fixed center height
    const baseGroundY = height - 120;
    const offset = altitude * 0.4;
    return Math.max(height / 2, baseGroundY - offset);
  };

  const drawRocketSprite = (ctx, x, y) => {
    // Rocket Width & Height dimensions
    const w = 24;
    const h = 75;

    // Crashing Explosion rendering
    if (flightStatus === 'crashed') {
      ctx.fillStyle = '#EF4444';
      ctx.beginPath();
      ctx.arc(x, y + 20, 40, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = '#F59E0B';
      ctx.beginPath();
      ctx.arc(x, y + 20, 25, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = '#FFFF00';
      ctx.beginPath();
      ctx.arc(x + 5, y + 15, 12, 0, Math.PI * 2);
      ctx.fill();
      return;
    }

    // Rocket Body Nose Cone
    ctx.fillStyle = '#E5E7EB'; // Sleek white body
    ctx.beginPath();
    ctx.moveTo(x, y - h/2);
    ctx.lineTo(x + w/2, y - h/4);
    ctx.lineTo(x - w/2, y - h/4);
    ctx.closePath();
    ctx.fill();

    // Nose Cone Red Tip
    ctx.fillStyle = '#EF4444';
    ctx.beginPath();
    ctx.moveTo(x, y - h/2);
    ctx.lineTo(x + w/3, y - h/3);
    ctx.lineTo(x - w/3, y - h/3);
    ctx.closePath();
    ctx.fill();

    // Main Cylindrical Body Tank
    ctx.fillStyle = '#F3F4F6';
    ctx.fillRect(x - w/2, y - h/4, w, h * 0.6);

    // Fuel Tank Stripe Decoration
    ctx.fillStyle = '#3B82F6';
    ctx.fillRect(x - w/2, y - 5, w, 6);

    // Window
    ctx.fillStyle = '#1F2937';
    ctx.beginPath();
    ctx.arc(x, y - 10, 5, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillStyle = '#9CA3AF';
    ctx.beginPath();
    ctx.arc(x + 1.5, y - 11.5, 2, 0, Math.PI * 2);
    ctx.fill();

    // Left Fin
    ctx.fillStyle = '#EF4444';
    ctx.beginPath();
    ctx.moveTo(x - w/2, y + h/6);
    ctx.lineTo(x - w, y + h/3);
    ctx.lineTo(x - w/2, y + h/3);
    ctx.closePath();
    ctx.fill();

    // Right Fin
    ctx.beginPath();
    ctx.moveTo(x + w/2, y + h/6);
    ctx.lineTo(x + w, y + h/3);
    ctx.lineTo(x + w/2, y + h/3);
    ctx.closePath();
    ctx.fill();

    // Engine Nozzle
    ctx.fillStyle = '#4B5563';
    ctx.beginPath();
    ctx.moveTo(x - w/4, y + h/3);
    ctx.lineTo(x + w/4, y + h/3);
    ctx.lineTo(x + w/3, y + h/2.4);
    ctx.lineTo(x - w/3, y + h/2.4);
    ctx.closePath();
    ctx.fill();

    // Small active thrust flame glow (base flame beneath nozzle)
    if (isRunning && currentFuel > 0) {
      const pulse = Math.sin(Date.now() * 0.05) * 5;
      const flameHeight = 25 + pulse;
      
      // Outer bright orange flame
      ctx.fillStyle = '#F97316';
      ctx.beginPath();
      ctx.moveTo(x - w/4, y + h/2.3);
      ctx.quadraticCurveTo(x, y + h/2.3 + flameHeight, x + w/4, y + h/2.3);
      ctx.closePath();
      ctx.fill();

      // Inner yellow core
      ctx.fillStyle = '#FDE047';
      ctx.beginPath();
      ctx.moveTo(x - w/6, y + h/2.3);
      ctx.quadraticCurveTo(x, y + h/2.3 + flameHeight * 0.6, x + w/6, y + h/2.3);
      ctx.closePath();
      ctx.fill();
    }
  };

  // Check Mission Success Criteria
  const activeMission = MISSIONS.find(m => m.id === activeMissionId);
  const isMissionPassed = activeMission && altitude >= activeMission.targetAltitude && selectedPlanetKey === activeMission.planet;

  return (
    <div className="min-h-screen bg-slate-950 text-slate-100 flex flex-col font-sans selection:bg-indigo-500 selection:text-white">
      {/* HEADER NAVBAR */}
      <header className="border-b border-slate-800 bg-slate-900/80 backdrop-blur px-6 py-4 flex flex-wrap gap-4 items-center justify-between sticky top-0 z-50">
        <div className="flex items-center gap-3">
          <div className="bg-gradient-to-tr from-indigo-500 to-rose-500 p-2 rounded-xl text-white shadow-lg shadow-indigo-500/20">
            <Rocket className="h-6 w-6 animate-pulse" />
          </div>
          <div>
            <h1 className="text-xl font-bold bg-gradient-to-r from-white via-slate-200 to-indigo-400 bg-clip-text text-transparent">
              Solaris Flight Dynamics
            </h1>
            <p className="text-xs text-slate-400">Interplanetary Rocket Thrust & Drag Simulator</p>
          </div>
        </div>

        <div className="flex items-center gap-2 text-xs bg-slate-800/60 p-1.5 rounded-lg border border-slate-700/50">
          <span className="px-2 py-1 text-slate-300 font-medium">Physics Mode:</span>
          <span className="px-2 py-0.5 rounded bg-indigo-500/20 text-indigo-300 border border-indigo-500/30">Relativistic Drag</span>
          <span className="px-2 py-0.5 rounded bg-emerald-500/20 text-emerald-300 border border-emerald-500/30">Barometric Gravity Drop</span>
        </div>
      </header>

      {/* CORE BODY GRID */}
      <main className="flex-1 max-w-[1700px] w-full mx-auto p-4 lg:p-6 grid grid-cols-1 lg:grid-cols-12 gap-6">
        
        {/* LEFT COLUMN: CONTROLS & PLANETS (4 COLS) */}
        <section className="lg:col-span-4 flex flex-col gap-6">
          
          {/* PLANET SELECTOR */}
          <div className="bg-slate-900 rounded-2xl p-5 border border-slate-800 shadow-xl flex flex-col gap-4">
            <div className="flex items-center gap-2">
              <Globe className="h-5 w-5 text-indigo-400" />
              <h2 className="text-lg font-semibold text-slate-100">Select Launch Site</h2>
            </div>
            
            <div className="grid grid-cols-3 gap-2">
              {Object.keys(PLANETS).map((key) => {
                const active = selectedPlanetKey === key;
                return (
                  <button
                    key={key}
                    onClick={() => {
                      setSelectedPlanetKey(key);
                      resetSimulation();
                    }}
                    className={`p-2.5 rounded-xl text-center border transition-all flex flex-col items-center justify-between ${
                      active 
                        ? 'bg-gradient-to-b from-indigo-900/50 to-indigo-950/80 border-indigo-500 text-indigo-200 ring-2 ring-indigo-500/20' 
                        : 'bg-slate-800/40 hover:bg-slate-800 border-slate-700/60 text-slate-300'
                    }`}
                  >
                    <span className="text-xs font-bold uppercase tracking-wider block mb-1">
                      {PLANETS[key].name.split(' ')[0]}
                    </span>
                    <span className="text-[10px] text-slate-400 block">
                      {PLANETS[key].gravity} m/s²
                    </span>
                  </button>
                );
              })}
            </div>

            {/* Selected Planet Stats */}
            <div className="mt-2 bg-slate-950 rounded-xl p-4 border border-slate-800/60 flex flex-col gap-3">
              <div className="flex items-center justify-between">
                <span className="text-sm font-semibold text-indigo-300 flex items-center gap-1.5">
                  <span className="w-2.5 h-2.5 rounded-full" style={{ backgroundColor: planet.color }} />
                  {planet.name} Surface Details
                </span>
                <span className="text-xs text-indigo-400 italic font-mono">
                  Esc: {Math.round(planet.escapeVelocity / 1000 * 10) / 10} km/s
                </span>
              </div>
              <p className="text-xs text-slate-300 leading-relaxed">{planet.description}</p>
              
              <div className="grid grid-cols-2 gap-2 mt-1 pt-3 border-t border-slate-900">
                <div>
                  <span className="text-[10px] text-slate-400 block uppercase font-semibold">Gravity</span>
                  <span className="text-sm font-mono text-slate-200">{planet.gravity} m/s²</span>
                </div>
                <div>
                  <span className="text-[10px] text-slate-400 block uppercase font-semibold">Surface Air Density</span>
                  <span className="text-sm font-mono text-slate-200">{planet.atmosphereDensity} kg/m³</span>
                </div>
              </div>

              <div className="bg-amber-950/20 rounded-lg p-2.5 border border-amber-500/20 text-[11px] text-amber-200 flex gap-2 items-start">
                <Info className="h-4 w-4 text-amber-400 shrink-0 mt-0.5" />
                <span><strong className="text-amber-300">Takeoff Rule:</strong> {planet.fact}</span>
              </div>
            </div>
          </div>

          {/* ROCKET BUILDER CONTROLS */}
          <div className="bg-slate-900 rounded-2xl p-5 border border-slate-800 shadow-xl flex flex-col gap-4">
            <div className="flex items-center justify-between">
              <div className="flex items-center gap-2">
                <Compass className="h-5 w-5 text-rose-400" />
                <h2 className="text-lg font-semibold text-slate-100">Rocket Configurator</h2>
              </div>
              
              {/* Preset vs Custom Select */}
              <div className="flex bg-slate-950 p-1 rounded-lg border border-slate-800">
                <button 
                  onClick={() => setActiveTab('presets')}
                  className={`px-3 py-1 text-xs font-medium rounded-md transition ${activeTab === 'presets' ? 'bg-indigo-500 text-white' : 'text-slate-400 hover:text-slate-200'}`}
                >
                  Presets
                </button>
                <button 
                  onClick={() => setActiveTab('advanced')}
                  className={`px-3 py-1 text-xs font-medium rounded-md transition ${activeTab === 'advanced' ? 'bg-indigo-500 text-white' : 'text-slate-400 hover:text-slate-200'}`}
                >
                  Custom
                </button>
              </div>
            </div>

            {/* PRESETS TAB */}
            {activeTab === 'presets' && (
              <div className="flex flex-col gap-2">
                {PRESETS.map((preset, idx) => (
                  <button
                    key={idx}
                    onClick={() => applyPreset(preset, idx)}
                    className={`p-3 rounded-xl border text-left transition-all ${
                      selectedPreset === idx 
                        ? 'bg-slate-800/80 border-indigo-500 shadow' 
                        : 'bg-slate-950/40 hover:bg-slate-800 border-slate-800'
                    }`}
                  >
                    <div className="flex justify-between items-center mb-1">
                      <span className="text-xs font-bold text-slate-100">{preset.name}</span>
                      <span className="text-[10px] px-2 py-0.5 rounded bg-indigo-500/10 text-indigo-300 border border-indigo-500/20 font-mono">
                        {preset.thrust} kN / {preset.isp}s
                      </span>
                    </div>
                    <p className="text-[11px] text-slate-400 line-clamp-1">{preset.description}</p>
                  </button>
                ))}
              </div>
            )}

            {/* CUSTOM / ADVANCED TAB */}
            {activeTab === 'advanced' && (
              <div className="flex flex-col gap-4">
                {/* Dry Mass */}
                <div>
                  <div className="flex justify-between text-xs mb-1">
                    <span className="text-slate-400">Dry Mass (Chassis / Payload)</span>
                    <span className="font-mono text-slate-200 font-bold">{(dryMass).toLocaleString()} kg</span>
                  </div>
                  <input
                    type="range"
                    min="500"
                    max="100000"
                    step="500"
                    value={dryMass}
                    onChange={(e) => {
                      setDryMass(Number(e.target.value));
                      setSelectedPreset(-1);
                    }}
                    className="w-full accent-indigo-500 h-1.5 bg-slate-800 rounded-lg cursor-pointer"
                  />
                </div>

                {/* Fuel Capacity */}
                <div>
                  <div className="flex justify-between text-xs mb-1">
                    <span className="text-slate-400">Fuel Mass (Propellants)</span>
                    <span className="font-mono text-slate-200 font-bold">{(fuelMass).toLocaleString()} kg</span>
                  </div>
                  <input
                    type="range"
                    min="2000"
                    max="1000000"
                    step="2000"
                    value={fuelMass}
                    onChange={(e) => {
                      setFuelMass(Number(e.target.value));
                      setSelectedPreset(-1);
                    }}
                    className="w-full accent-indigo-500 h-1.5 bg-slate-800 rounded-lg cursor-pointer"
                  />
                </div>

                {/* Engine Max Thrust */}
                <div>
                  <div className="flex justify-between text-xs mb-1">
                    <span className="text-slate-400">Engine Max Thrust</span>
                    <span className="font-mono text-slate-200 font-bold">{(engineThrust).toLocaleString()} kN</span>
                  </div>
                  <input
                    type="range"
                    min="50"
                    max="20000"
                    step="50"
                    value={engineThrust}
                    onChange={(e) => {
                      setEngineThrust(Number(e.target.value));
                      setSelectedPreset(-1);
                    }}
                    className="w-full accent-indigo-500 h-1.5 bg-slate-800 rounded-lg cursor-pointer"
                  />
                </div>

                {/* Fuel Efficiency Specific Impulse */}
                <div>
                  <div className="flex justify-between text-xs mb-1">
                    <span className="text-slate-400">Specific Impulse (Isp)</span>
                    <span className="font-mono text-slate-200 font-bold">{isp} seconds</span>
                  </div>
                  <input
                    type="range"
                    min="150"
                    max="900"
                    step="10"
                    value={isp}
                    onChange={(e) => {
                      setIsp(Number(e.target.value));
                      setSelectedPreset(-1);
                    }}
                    className="w-full accent-indigo-500 h-1.5 bg-slate-800 rounded-lg cursor-pointer"
                  />
                </div>

                {/* Aerodynamic settings */}
                <div className="grid grid-cols-2 gap-4 pt-3 border-t border-slate-800">
                  <div>
                    <label className="text-[10px] text-slate-400 uppercase tracking-wider block mb-1">Drag Coef ($C_D$)</label>
                    <select
                      value={dragCoef}
                      onChange={(e) => setDragCoef(Number(e.target.value))}
                      className="w-full bg-slate-950 border border-slate-800 rounded-lg px-2 py-1 text-xs text-slate-200 font-mono focus:outline-none focus:border-indigo-500"
                    >
                      <option value="0.1">0.10 (Ultra Sleek)</option>
                      <option value="0.3">0.30 (Standard Cone)</option>
                      <option value="0.5">0.50 (Bulky Head)</option>
                      <option value="0.8">0.80 (Unshielded)</option>
                    </select>
                  </div>
                  <div>
                    <label className="text-[10px] text-slate-400 uppercase tracking-wider block mb-1">Area ($A$)</label>
                    <select
                      value={crossSection}
                      onChange={(e) => setCrossSection(Number(e.target.value))}
                      className="w-full bg-slate-950 border border-slate-800 rounded-lg px-2 py-1 text-xs text-slate-200 font-mono focus:outline-none focus:border-indigo-500"
                    >
                      <option value="3.14">3.14 m² (Slim)</option>
                      <option value="7.0">7.00 m² (Normal)</option>
                      <option value="15.0">15.0 m² (Heavy Stage)</option>
                    </select>
                  </div>
                </div>
              </div>
            )}
          </div>
        </section>

        {/* MIDDLE COLUMN: CANVAS GRAPHICS & HUD SIMULATOR (5 COLS) */}
        <section className="lg:col-span-5 flex flex-col gap-6">
          
          {/* SIMULATOR WINDOW PANEL */}
          <div className="bg-slate-900 rounded-2xl border border-slate-800 shadow-xl overflow-hidden flex flex-col relative h-[560px]">
            {/* Top HUD overlay */}
            <div className="absolute top-0 inset-x-0 bg-gradient-to-b from-slate-950/80 to-transparent p-4 flex justify-between items-start pointer-events-none z-10">
              <div className="flex flex-col gap-1">
                <span className="text-[10px] font-bold tracking-wider text-indigo-400 uppercase">
                  SIMULATING FLIGHT ON {planet.name.toUpperCase()}
                </span>
                <span className={`text-xs font-semibold px-2 py-0.5 rounded-full inline-block w-fit ${
                  flightStatus === 'grounded' ? 'bg-slate-800 text-slate-300' :
                  flightStatus === 'launching' ? 'bg-rose-500 text-white animate-pulse' :
                  flightStatus === 'escaped' ? 'bg-emerald-500 text-white' :
                  flightStatus === 'crashed' ? 'bg-red-500 text-white' : 'bg-indigo-500 text-white'
                }`}>
                  STATUS: {flightStatus.toUpperCase()}
                </span>
              </div>

              {/* TWR instant indicator */}
              <div className="bg-slate-950/90 border border-slate-800 rounded-xl px-3 py-1.5 text-right flex flex-col">
                <span className="text-[9px] text-slate-400 font-medium">REAL-TIME TWR</span>
                <span className={`text-base font-bold font-mono ${currentTWR >= 1.0 ? 'text-emerald-400' : 'text-red-400'}`}>
                  {currentTWR.toFixed(2)}
                </span>
                <span className="text-[9px] text-slate-500">Need &gt; 1.0</span>
              </div>
            </div>

            {/* The canvas */}
            <div className="flex-1 w-full bg-slate-950 relative">
              <canvas 
                ref={canvasRef} 
                width={500} 
                height={460} 
                className="w-full h-full block"
              />

              {/* Floating altitude gauge overlay right */}
              <div className="absolute right-4 top-24 bottom-16 w-12 bg-slate-950/70 border border-slate-800 rounded-xl py-4 px-1 flex flex-col items-center justify-between pointer-events-none text-[10px] text-slate-400 font-mono">
                <span>100k+</span>
                <div className="flex-1 w-1 bg-slate-800 rounded my-2 relative overflow-hidden">
                  <div 
                    className="absolute bottom-0 inset-x-0 bg-indigo-500 transition-all duration-100"
                    style={{ height: `${Math.min(100, (altitude / 30000) * 100)}%` }}
                  />
                </div>
                <span>0m</span>
              </div>
            </div>

            {/* Bottom Physics Real-time telemetry dashboard */}
            <div className="bg-slate-950 border-t border-slate-800 p-4 grid grid-cols-4 gap-2 text-center">
              <div className="bg-slate-900/50 p-2 rounded-xl border border-slate-900">
                <span className="text-[9px] text-slate-400 uppercase block mb-0.5">Altitude</span>
                <span className="text-sm font-bold font-mono text-indigo-300">
                  {altitude >= 1000 ? `${(altitude/1000).toFixed(2)} km` : `${Math.round(altitude)} m`}
                </span>
              </div>
              <div className="bg-slate-900/50 p-2 rounded-xl border border-slate-900">
                <span className="text-[9px] text-slate-400 uppercase block mb-0.5">Velocity</span>
                <span className="text-sm font-bold font-mono text-emerald-300">
                  {Math.round(velocity)} m/s
                </span>
              </div>
              <div className="bg-slate-900/50 p-2 rounded-xl border border-slate-900">
                <span className="text-[9px] text-slate-400 uppercase block mb-0.5">Fuel Remaining</span>
                <span className="text-sm font-bold font-mono text-amber-400">
                  {Math.round((currentFuel / fuelMass) * 100)}%
                </span>
              </div>
              <div className="bg-slate-900/50 p-2 rounded-xl border border-slate-900">
                <span className="text-[9px] text-slate-400 uppercase block mb-0.5">Mass Ratio</span>
                <span className="text-sm font-bold font-mono text-slate-300">
                  {(totalMass / dryMass).toFixed(1)}x
                </span>
              </div>
            </div>

            {/* Physics warning overlays */}
            {flightStatus === 'crashed' && (
              <div className="absolute inset-0 bg-slate-950/90 flex flex-col justify-center items-center text-center p-6 z-20">
                <ShieldAlert className="h-16 w-16 text-red-500 mb-2 animate-bounce" />
                <h3 className="text-xl font-bold text-red-400">MISSION FAILURE</h3>
                <p className="text-sm text-slate-300 mt-2 max-w-sm">{crashReason}</p>
                <button 
                  onClick={resetSimulation}
                  className="mt-6 px-5 py-2.5 bg-red-600 hover:bg-red-500 text-white font-semibold rounded-xl text-xs transition shadow-lg shadow-red-500/20 flex items-center gap-2"
                >
                  <RefreshCw className="h-3.5 w-3.5" /> Re-Assemble Rocket
                </button>
              </div>
            )}

            {flightStatus === 'escaped' && (
              <div className="absolute inset-0 bg-slate-950/95 flex flex-col justify-center items-center text-center p-6 z-20">
                <Award className="h-16 w-16 text-emerald-400 mb-2 animate-bounce" />
                <h3 className="text-xl font-bold text-emerald-400">ESCAPE VELOCITY ACHIEVED</h3>
                <p className="text-sm text-slate-300 mt-2 max-w-sm">
                  Congratulations! Your rocket reached a velocity of <strong className="text-emerald-300 font-mono">{Math.round(velocity)} m/s</strong>, breaking past {planet.name}'s escape threshold of {planet.escapeVelocity} m/s.
                </p>
                <button 
                  onClick={resetSimulation}
                  className="mt-6 px-5 py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white font-semibold rounded-xl text-xs transition shadow-lg shadow-emerald-500/20 flex items-center gap-2"
                >
                  <RefreshCw className="h-3.5 w-3.5" /> Launch Again
                </button>
              </div>
            )}
          </div>

          {/* SIMULATOR CONTROLS */}
          <div className="bg-slate-900 rounded-2xl p-4 border border-slate-800 shadow-xl flex gap-3">
            {isRunning ? (
              <button
                onClick={() => setIsRunning(false)}
                className="flex-1 py-3 bg-rose-600 hover:bg-rose-500 text-white font-semibold rounded-xl transition flex items-center justify-center gap-2 shadow-lg shadow-rose-600/15"
              >
                <Pause className="h-4 w-4 fill-current" /> Cut Throttle (Kill Ignition)
              </button>
            ) : (
              <button
                onClick={() => {
                  if (flightStatus === 'crashed' || flightStatus === 'escaped') {
                    resetSimulation();
                  }
                  setIsRunning(true);
                }}
                disabled={currentFuel <= 0 && flightStatus === 'grounded'}
                className="flex-1 py-3 bg-emerald-600 hover:bg-emerald-500 text-white font-semibold rounded-xl transition flex items-center justify-center gap-2 disabled:opacity-40 disabled:pointer-events-none shadow-lg shadow-emerald-600/15"
              >
                <Play className="h-4 w-4 fill-current animate-pulse" /> Launch Ignition (Engage Thrust)
              </button>
            )}

            <button
              onClick={resetSimulation}
              className="px-4 bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 rounded-xl transition flex items-center justify-center"
              title="Reset Flight"
            >
              <RefreshCw className="h-4 w-4" />
            </button>
          </div>
        </section>

        {/* RIGHT COLUMN: REAL-TIME PHYSICS, MISSIONS & COMPARISON (3 COLS) */}
        <section className="lg:col-span-3 flex flex-col gap-6">
          
          {/* CAMPAIGN CHALLENGES */}
          <div className="bg-slate-900 rounded-2xl p-5 border border-slate-800 shadow-xl flex flex-col gap-4">
            <div className="flex items-center gap-2 justify-between">
              <div className="flex items-center gap-2">
                <Award className="h-5 w-5 text-emerald-400" />
                <h2 className="text-base font-bold text-slate-100">Sim Missions</h2>
              </div>
              <span className="text-[10px] bg-slate-950 px-2 py-0.5 rounded text-indigo-400 border border-slate-800">
                Objectives
              </span>
            </div>

            <div className="flex flex-col gap-3">
              {MISSIONS.map((mission) => {
                const isCurrent = activeMissionId === mission.id;
                const matchesPlanet = selectedPlanetKey === mission.planet;
                const passed = altitude >= mission.targetAltitude && matchesPlanet;

                return (
                  <div
                    key={mission.id}
                    onClick={() => startMission(mission)}
                    className={`p-3.5 rounded-xl border text-left transition cursor-pointer relative overflow-hidden ${
                      isCurrent 
                        ? 'bg-indigo-950/30 border-indigo-500/70' 
                        : 'bg-slate-950/30 hover:bg-slate-800/40 border-slate-850'
                    }`}
                  >
                    {passed && (
                      <div className="absolute top-2 right-2 flex items-center gap-1 text-[10px] font-bold text-emerald-400 bg-emerald-500/10 px-2 py-0.5 rounded border border-emerald-500/20">
                        PASSED <Sparkles className="h-2.5 w-2.5" />
                      </div>
                    )}
                    
                    <div className="flex items-center gap-1.5 mb-1.5">
                      <span className="w-2 h-2 rounded-full" style={{ backgroundColor: PLANETS[mission.planet].color }} />
                      <span className="text-xs font-bold text-slate-100">{mission.title}</span>
                    </div>
                    
                    <p className="text-[11px] text-slate-300 leading-normal mb-2">{mission.description}</p>
                    
                    <div className="flex items-center justify-between text-[10px] text-slate-400 border-t border-slate-800/80 pt-2">
                      <span>Target: <strong className="text-indigo-300 font-mono">{(mission.targetAltitude).toLocaleString()}m</strong></span>
                      <span className="capitalize">{mission.planet}</span>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>

          {/* TELEMETRY HUD DETAILS */}
          <div className="bg-slate-900 rounded-2xl p-5 border border-slate-800 shadow-xl flex flex-col gap-4">
            <div className="flex items-center gap-2">
              <Gauge className="h-5 w-5 text-indigo-400" />
              <h2 className="text-base font-bold text-slate-100">Telemetry Analysis</h2>
            </div>

            <div className="flex flex-col gap-3 text-xs">
              <div className="flex justify-between items-center py-2 border-b border-slate-850">
                <span className="text-slate-400">Total Wet Mass</span>
                <span className="font-mono text-slate-200">{(totalMass).toLocaleString()} kg</span>
              </div>
              
              <div className="flex justify-between items-center py-2 border-b border-slate-850">
                <span className="text-slate-400">Atmosphere Density</span>
                <span className="font-mono text-slate-200">
                  {planet.atmosphereDensity > 0 
                    ? `${(planet.atmosphereDensity * Math.exp(-altitude / planet.scaleHeight)).toFixed(4)} kg/m³` 
                    : '0.0000 (Vacuum)'}
                </span>
              </div>

              <div className="flex justify-between items-center py-2 border-b border-slate-850">
                <span className="text-slate-400">Drag Force Opposing</span>
                <span className="font-mono text-rose-400">
                  {Math.round(0.5 * (planet.atmosphereDensity * Math.exp(-altitude / planet.scaleHeight)) * Math.pow(velocity, 2) * dragCoef * crossSection).toLocaleString()} N
                </span>
              </div>

              <div className="flex justify-between items-center py-2 border-b border-slate-850">
                <span className="text-slate-400">Gravity Drop-Off</span>
                <span className="font-mono text-slate-200">
                  {(planet.gravity * Math.pow(planet.radius / (planet.radius + altitude), 2)).toFixed(2)} m/s²
                </span>
              </div>

              <div className="flex justify-between items-center py-2 border-b border-slate-850">
                <span className="text-slate-400">Fuel Burn Duration</span>
                <span className="font-mono text-slate-200">
                  {Math.round(fuelMass / ((engineThrust * 1000) / (isp * 9.81)))} seconds
                </span>
              </div>

              <div className="flex justify-between items-center py-2">
                <span className="text-slate-400">Max Flight Speed</span>
                <span className="font-mono text-emerald-400">{maxVelocity.toLocaleString()} m/s</span>
              </div>
            </div>
          </div>

          {/* EDUCATIONAL BRIEF */}
          <div className="bg-slate-900 rounded-2xl p-5 border border-slate-800 shadow-xl text-xs flex flex-col gap-3">
            <h3 className="font-bold text-slate-200 flex items-center gap-1.5">
              <Info className="h-4 w-4 text-indigo-400" />
              Thrust-to-Weight Equation
            </h3>
            <p className="text-slate-300 leading-relaxed">
              For a rocket to accelerate upwards, its vertical thrust must exceed local gravity ($T > m \cdot g$).
            </p>
            <div className="bg-slate-950 p-2.5 rounded-xl border border-slate-850 text-center font-mono text-indigo-300">
              $$\text{TWR} = \frac{\text{Thrust (N)}}{\text{Mass (kg)} \cdot \text{Gravity} (m/s^2)}$$
            </div>
            <p className="text-slate-300 leading-relaxed">
              If <strong className="text-rose-400">TWR &lt; 1</strong>, the rocket will burn fuel on the pad but fail to lift off. Drag further reduces acceleration, requiring even higher ratios.
            </p>
          </div>

        </section>

      </main>

      {/* FOOTER */}
      <footer className="border-t border-slate-800 bg-slate-900/40 py-6 text-center text-xs text-slate-400 mt-auto">
        <p>© 2026 Solaris Spaceflight Simulator. Designed for deployment on GitHub Pages and Vercel.</p>
        <p className="mt-1">Built with high-fidelity real planetary gravity profiles and barometric scale height formulas.</p>
      </footer>
    </div>
  );
}
