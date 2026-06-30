---
name: presentation-threejs-mastery
description: Expert guidelines for manipulating Three.js (React Three Fiber) in the Gossip Garden presentation. Covers camera rigs, globe geometry, parallax, and particle rendering. Trigger whenever requested to edit 3D logic.
---

# Presentation Three.js (React Three Fiber) Mastery

This skill defines the strict parameters and techniques for manipulating the 3D environment in the Gossip Garden Next.js presentation (`gossip-garden-presentation/components/three/`).

## 1. Architectural Philosophy
- **Framework**: We use `@react-three/fiber` (R3F) and `@react-three/drei`. Do NOT write raw imperative Three.js code inside `useEffect` unless absolutely necessary. Rely on declarative JSX `<mesh>`, `<group>`, `<Canvas>`, etc.
- **Performance**: We are rendering on laptops and possibly mobile devices. Always use `useMemo` for geometries, materials, and complex math (like particle arcs).
- **Aesthetic**: The 3D world must match the "Crayon Storybook" 2D aesthetic. Avoid photorealism. We use baked textures, basic materials for outlines, and low-roughness standard materials for the globe.

## 2. Camera Rig & Navigation
The camera is never controlled by traditional orbit controls. It is fully scripted and reacts to two things:
1. **The current `Beat` (`scene` key)**: We zoom in and out by interpolating `camera.position.z` in the `CameraRig` component. 
2. **Mouse Parallax**: We track the user's mouse via R3F's `pointer` state and slightly offset the camera in X and Y to give a pseudo-3D feel.

**Rules for Camera Modification:**
- **DO NOT** use CSS `scale` on the `<Canvas>` or its wrapper to simulate zooming. This breaks the internal R3F raycaster and pointer coordinates, causing elements to fly off-center.
- **ALWAYS** modify the `target` Z-distance inside the `switch (scene)` statement in `CameraRig`.
- **Damping**: Always use `THREE.MathUtils.damp()` inside `useFrame` to ensure smooth, frame-rate independent camera movements. Do not use direct linear interpolation (`lerp`) without delta time.

## 3. The Globe (Earth)
- The globe is composed of two meshes:
  1. The primary textured sphere (`meshStandardMaterial`).
  2. A slightly larger, inverted sphere (`THREE.BackSide`) using a brown `meshBasicMaterial` to create the "crayon outline" effect natively in 3D.
- To modify the rotation or wobble of the planet, adjust the `useFrame` hook inside `WorldGroup`. Keep the movement organic and slow.

## 4. Particles and Connections (Roots)
- **Roots**: Drawn using `BufferGeometry` and `setDrawRange` to simulate them "growing" from one node to another. 
- **Data Particles**: Handled via a single `THREE.Points` instance with a large `Float32Array` for positions. This is highly optimized. We calculate the position of each particle along a predefined arc using the `arcPoints` utility.
- When creating new visual effects, avoid instantiating thousands of individual meshes. Use `InstancedMesh` or `Points` for performance.

## 5. HTML Overlays
- When pinning HTML to 3D coordinates (like the `KnowledgeBubbles`), use the `<Html>` component from `@react-three/drei`.
- Ensure they are styled using the existing CSS classes from `globals.css` (e.g., `.crayon-card`, `.crayon-text`) to seamlessly blend with the 2D UI.
