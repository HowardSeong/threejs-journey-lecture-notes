# 강의 목표
- Shader가 무엇인지 설명할 수 있다.
- 간단한 Shader를 만들어 보고 이해한다.
- Syntax를 학습한다.
- Shader 작성 연습을 한다.

## Shader란?
- WebGL의 주요 요소 중 하나로, 네이티브 WebGL을 사용할 때 가장 먼저 배워야 한다.
- GLSL 언어로 작성된 프로그램으로 GPU에 전달된다.
- Geometry의 Vertex 위치를 지정한다.
- Geometry 상에 보여지는 Pixel들의 색상들을 결정한다.
  - 렌더 상의 각각의 점들은 화면 상의 "Pixel"과 완전히 일치하지 않기 때문에, "Pixel"은 정확하지 않다.
  - 따라서, "Fragment" 개념을 사용한다. "Fragment"는 Pixel과 비슷하지만, 렌더 상에서 활용하는 개념이다.
- Shader로 다양한 정보들을 전달할 수 있다. GPU는 Shader의 지시대로 다음의 정보들을 처리한다.
  - Vertices coordinates
  - Mesh transformation
  - Information about the Camera
  - Colors
  - Textures
  - Lights
  - Fog
  - Etc
- 두 종류의 Shader가 존재한다.

### Vertex Shader
- Geometry의 각 Vertex 위치를 지정한다.
- 다음의 순서로 Vertex Shader가 동작한다.
  1. Vertex Shader를 작성한다.
  2. 정점들의 좌표 정보, Mesh 변환 정보, 카메라 정보 등과 함께 Shader를 GPU로 전달한다.
  3. GPU는 Shader의 지시에 따라, 정점들을 렌더 상에 위치시킨다.
- 동일한 Vertex Shader가 각각의 정점들에 사용된다.
- 정점의 위치는 각 정점들마다 다른데, 이런 형태의 데이터를 'attribute'라고 한다.
- Mesh의 위치는 모든 정점에 대해 같은데, 이런 형태의 데이터를 'uniform'이라고 한다.
- 정점들이 Vertex Shader에 의해 위치하고 나면, GPU는 Geometry의 어떤 pixel들이 보여지는지 알 수 있고, 이 것들은 Fragment Shader로 전달된다.

### Fragment Shader
- Geometry 상에 보여지는 Pixel들의 색상들을 결정한다.
- 동일한 Fragment Shader가 geometry의 각각의 보여지는 fragement들에 사용된다.
- 다음의 순서로 Fragment Shader가 동작한다.
  1. Fragment Shader를 작성한다.
  2. 색상과 같은 정보와 함께, Shader를 GPU로 전달한다.
  3. GPU는 Shader의 지시에 따라, fragment들의 색상을 결정한다.
- Fragment에 'uniform' 데이터를 전달할 수 있다.
- Vertex에서 Fragment로 전달되는 데이터를 'varying'이라고 하며, 이 값은 정점들 사이에 보간(Interpolate)된다.

### Summary
- `vertex shader`는 render 상에 점정들을 위치시킨다.
- `fragment shader`는 geometry에서 보여지는 fragment(또는 pixel)들의 색상을 결정한다.
- `fragement shader`는 `vertex shader` 이후에 동작한다.
- 위치와 같이 각각의 정점들 사이에 다른 정보들을 `attributes`라고 하며, `vertex shader`에서만 사용된다.
- 정점들이나 fragment들 사이에 변하지 않는 정보들을 `uniforms`라고 하며, `vertex shader`와 `fragment shader`에서 모두 사용할 수 있다.
- `varyings`을 통해 `vertex shader`에서 `fragment shader`로 정보를 전달할 수 있다.
- `varyings`들은 정점들 사이에 보간(Interpolate)된다.

### 왜 Shader를 작성하는가?
- Three.js의 material들은 제한적이다.
- Shader는 매우 단순하고, 성능이 우수하다.
- Custom post-precessing을 추가할 수 있다.

## `RawShaderMaterial`로 첫 번째 Shader 만들기
Three.js 에서 제공되는 두 가지 클래스를 사용할 수 있다. - `ShaderMaterial`, `RawShaderMaterial`
- `ShaderMaterial`는 일부 미리 작성된 코드가 자동으로 삽입된다.
- `RawShaderMaterial`는 아무것도 가지지 않는다.
- Three.js에서 제공되는 Material의 기본 속성들을 사용할 수 있다.
  - `wireframe`, `side`, `transparent`, `flatShading`와 같은 속성값을 지정할 수 있다.
  - `map`, `alphaMap`, `opacity`, `color` 등은 동작하지 않고, 직접 개발해야 한다.

```typescript
const material = new THREE.RawShaderMaterial({
    vertexShader: `
        uniform mat4 projectionMatrix;
        uniform mat4 viewMatrix;
        uniform mat4 modelMatrix;

        attribute vec3 position;

        void main()
        {
          gl_Position = projectionMatrix * viewMatrix * modelMatrix * vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        precision mediump float;

        void main() 
        {
          gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
        }
    `,
});
```

### Shaders 파일 분리하기
아래와 같이 파일을 분리하여 저장한다.
- `shaders/${name}/vertex.glsl`
- `shaders/${name}/fragment.glsl`

Webpack Config를 통해 glsl 파일을 모듈로 로드할 수 있도록 한다.
`rules` 배열에 다음의 rule을 추가한다.<br>
`Webpack 5 부터는 Asset Modules를 활용하여 에셋을 로드한다.`
- Webpack < 5
```txt
{
    test: /\.(glsl|vs|fs|vert|frag)$/,
    exclude: /node_modules/,
    use: [
        'raw-loader
    ]
}
```
- Webpack >=  5
```txt
{
    test: /\.(glsl|vs|fs|vert|frag)$/,
    type: 'asset/source',
    generator:
    {
        filename: 'assets/images/[hash][ext]'
    }
}
```

```typescript
import materialVertexShader from 'shaders/material/vertex.glsl';
import materialFragmentShader from 'shaders/material/fragment.glsl';

const material = new THREE.RawShaderMaterial({
    vertexShader: materialVertexShader,
    fragmentShader: materialFragmentShader,
});
```

## GLSL (OpenGL Shading Language)

### 로깅
콘솔이 존재하지 않는다.

### 들여쓰기
들여쓰기는 필수가 아니다.

### 세미콜론
각각의 명령문에는 세미콜론이 반드시 필요하다.

### 변수
타입 언어로 변수의 타입을 명시해야하며, 다른 타입을 변수에 할당할 수 없다.<br>
산술 연산자를 사용할 수 있다.<br>
타입 변환이 가능하다.<br>
다양한 타입
- float : 소수
- int : 정수
- bool : true / false
- vec2
  - 2차원 벡터로 `x`, `y`를 저장한다.
    ```glsl
      vec2 foo = vec2(1.0, 2.0);
    ```
  - 빈 `vec2`는 에러가 발생한다.
  - 하나의 값은 모든 속성에 사용된다.
    ```glsl
      vec2 foo = vec2(0.0);
    ```
  - 속성은 `x`, `y`로 접근하여 변경할 수 있다.
    ```glsl
      vec2 foo = vec2(0.0);
      foo.x = 1.0;
      foo.y = 2.0;
    ```
  - `vec2`에 곱하기 연산을 하면, 각각의 속성에 모두 연산을 수행한다.
    ```glsl
      vec2 foo = vec2(1.0, 2.0);
      foo *= 2.0;
    ```
- vec3
  - 3차원 벡터로 `x`, `y`, `z`를 저장한다.
  - `r`, `g`, `b`로도 값을 지정할 수 있다. 색상값 지정에 활용한다.
    ```glsl
      vec3 purpleColor = vec3(0.0);
      purpleColor.r = 0.5;
      purpleColor.b = 1.0;
    ```
  - `vec2`를 이용해서 생성도 가능하다.
    ```glsl
      vec2 foo = vec2(1.0, 2.0);
      vec3 bar = vec3(foo, 3.0);
    ```
  - `vec2`를 생성할 수도 있다. (swizzle이라고 한다.)
    ```glsl
      vec3 foo = vec3(1.0, 2.0, 3.0);
      vec2 bar = foo.xy;
    ```
- vec4
  - 4차원 벡터로 `x`, `y`, `z`, `w`를 저장한다. (`w` 대신 `a`를 사용할 수 있다.)
- mat2
- mat3
- mat4
- sampler2D

### 함수
함수는 리턴 타입을 시작으로 작성한다.
```glsl
float loremIpsum()
{
    float a = 1.0;
    float b = 2.0;
    
    return a + b;
}
```
리턴 값이 없다면, `void`로 리턴 타입을 정의한다.<br>
파라미터에도 타입을 명시해야 한다.

### 네이티브 함수
빌트인 함수로 다음과 같은 것들이 있다.
- `sin`, `cos`, `max`, `min`, `pow`, `exp`, `mod`, `clamp`
- `cross`, `dot`, `mix`, `step`, `smoothstep`, `length`, `distance`, `reflect`, `refract`, `normalize`
- 참고할 만한 링크
  - [Sahderific](https://shaderific.com/glsl.html)
  - [Book of Shaders glossary](https://thebookofshaders.com/)

### Vertex Shader 이해하기
- `main` 함수는 자동으로 호출된다.
- `gl_Position`은 미리 정의된 변수로 직접 할당해야 한다. 화면 상의 정점의 위치를 지정한다.
- 아래 식은 `vec4`를 리턴한다.
```glsl
projectionMatrix * viewMatrix * modelMatrix * vec4(position, 1.0)
```
- `vec4`를 사용하는 이유는 `clip space` 때문이다.
  - 물체를 박스 안에서 배치하는 것과 같다.
  - `z`는 깊이를 나타낸다. (어떤 부분이 다른 부분 앞에 있는 지 알기 위함.)
  - `w`는 `perspective`를 담당한다.
- attribute: position
  - 각각의 정점마다 다른 값을 갖는다.
- uniform: matrix
  - 각각의 행령은 최종적으로 clip space의 좌표를 얻을 때까지 `position`을 변환한다.
  - 모든 정점에 대해 같은 연산을 수행하기 위해, `uniform`으로 정의한다.
  - 행렬 곱셈을 통해 `vec4`를 얻는다.
  - `modelMatrix`은 `Mesh(position, rotation, scale)`에 대한 변환을 적용한다.
  - `viewMatrix`는 `Camera(position, rotation, field of view, near, far)`에 대한 변환을 적용한다.
  - `porjectionsMatrix`는 좌표를 clip space 좌표로 변환한다.
- [좌표계 이해하기](https://learnopengl.com/Getting-started/Coordinate-Systems)
- `modelMatrix`와 `viewMatrix`를 합쳐서 `modelViewMatrix`로 표현할 수 있다.
  ```glsl
    gl_Position = porjectionMatrix * modelViewMatrix * vec4(position, 1.0);
  ```
- 각각을 분리하여 표현할 수도 있다.
  ```glsl
    vec4 modelPosition = modelMatrix * vec4(position, 1.0);
    // modelPosition.z += sin(modelPosition.x * 10.0) * 0.1;
    vec4 viewPosition = viewMatrix * modelPosition;
    vec4 projectedPosition = projectionMatrix * viewPosition;
    gl_Position = projectedPosition;
  ```

### Fragment Shader 이해하기
- `main` 함수는 자동으로 호출된다.
- `float` 값의 `precision`을 반드시 결정해야 한다.
  - `highp`: 성능 이슈가 있을 수 있고, 일부 기기에서 동작하지 않을 수 있다.
  - `mediump`: 보통 사용하는 값이다.
  - `lowp`: 정확도 부족으로 버그가 발생할 수 있다.
  - `ShaderMaterial`을 이용하면, 자동으로 조정된다.
- `gl_FragColor`은 미리 정의된 변수로 직접 할당해야 한다. Geometry 상에 보여지는 fragment의 색상을 지정한다.
  - `vec4`는 `r`, `g`, `b`, `a`를 갖는다.
  - alpha 값이 1.0 미만이라면, `transparent` 속성이 `true`이어야 한다.
    ```typescript
      const material = new THREE.RawShaderMaterial({
          vertexShader: materialVertexShader,
          fragmentShader: materialFragmentShader,
          transparent: true,
      });
    ```
    
### Vertex Shader, Fragment Shader 종합
- `attributes`
  - `BufferGeometry`에 직접 정의한 attributes를 추가할 수 있다.
  - `aRandom` attribute
    ```typescript
      const count = geometry.attributes.position.count;
      const randoms = new Float32Array(count);
      for (let i = 0; i < count; i++) {
          randoms[i] = Math.random();
      }
      geometry.setAttribute('aRandom', new THREE.BufferAttribute(randoms, 1));
    ```
    ```glsl
      // ...
      attribute float aRandom;
      void main()
      {
          // ...
          modelPosition.z += aRandom * 0.1;
          // ...
      }
    ```
- `varyings`
  - `aRandom`으로 fragments들의 색상을 지정하지만, fragment shader에서는 attributes를 사용할 수 없다.
  - vertex shader에서 fragment shader로 데이터를 전달할 때, `varyings`를 사용한다.
  - vertex shader 내부에, `vRandom` varying을 추가하고, `main` 함수를 업데이트 한다.
  - fragment shader 내부에도 동일한 이름으로 변수를 선언한다.
  - `varying`값은 정점들 사이에 보간(interpolate)된다.
    ```glsl
      // ...
      varying float vRandom;
      void main()
      {
          // ...
          vRandom = aRandom;
      }
    ```
    ```glsl
      precision mediump float;
      varying float vRandom;
      void main()
      {
        gl_FragColor = vec4(vRandom, vRandom * 0.5, 1.0, 1.0);
      }
    ```
- `uniforms`
  - 다음과 같은 경우에 사용할 수 있다.
    - 동일한 shader로 서로 다른 결과를 만들어 낼 수 있다.
    - 값들을 비틀 수 있다.
    - 값을 변화(animate)하게 할 수 있다.
  - vertex와 fragment shader 모두에 쓰일 수 있다.
  - `uniform` 속성으로 `material`에 uniform을 추가할 수 있다.
    ```typescript
      const material = new THREE.RawShaderMaterial({
          // ...
          uniforms:
          {
              uFrequency: { value: new THREE.Vector2(10, 5) }
          }
      });
    ```
    ```glsl
      // ...
      uniform vec2 uFrequency;
      // ...
      void main()
      {
          // ...
          modelPosition.z += sin(modelPosition.x * uFrequency.x) * 0.1;
          modelPosition.z += sin(modelPosition.y * uFrequency.y) * 0.1;
          // ...
      }
    ```
- texture 적용하기
  - texture의 픽셀 색상을 fragment shader에 적용하려면 `texture2D` 함수를 사용한다.
  - 첫 번째 인자는 texture, 두 번째 인자는 uv 좌표(texture 상의 좌표) 이다.
  - uv는 geometery 상에 정의된 attribute로 fragment shader에 적용하려면, varying으로 정의한다.
  ```typescript
    const material = new THREE.RawShaderMaterial({
        // ...
        uniforms:
        {
            uTexture: { value: flagTexture }
        }
    });
  ```
  ```glsl
    attribute vec2 uv;
    varying vec2 vUv;
    // ...
    void main()
    {
        // ...
        vUV = uv;
    }
  ```
  ```glsl
    uniform sampler2D uTexture;
    varying vec2 vUv;
    
    void main()
    {
        vec4 textureColor = texture2D(uTexture, vUv);
        gl_FragColor = textureColor;
    }
  ```
  
### ShaderMaterial
`ShaderMaterial`은 미리 작성된 uniforms, attributes, precision을 shader 코드에 추가한다.
