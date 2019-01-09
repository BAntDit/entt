
# Enttx

Compile-time and header-only [Entity-Component-System](https://en.wikipedia.org/wiki/Entity_component_system) library.

## Usage tutorial

### configuration

User application should provide specializations for following config templates:

1. EntityManagerConfig to define list of possible types of components and those storages.

```cpp
using entity_manager_config_t = enttx::EntityManagerConfig<
        easy_mp::type_list<
                Transform,
                Camera,
                DirectionalLight
            >,
        easy_mp::type_list<
                enttx::ComponentStorage<16, 128, Transform>,
                enttx::ComponentStorage<16, 128, Camera>,
                enttx::ComponentStorage<16, 128, DirectionalLight>
            >
    >;
```

2. SystemManagerConfig to define list of systems and update stages

```cpp
using system_manager_config_t = enttx::SystemManagerConfig<
        UPDATE_STAGES::STAGE_COUNT,
        entity_manager_config_t,
        easy_mp::type_list<
                CameraSystem,
                DirectionalLightSystem,
                TransformSystem
            >
    >;
```

### entities

### components

### entities and components management

EntityManager class provides all necessary functionality to manage entities and those classes

1. to create / destroy entity

```cpp
auto entity = entityManager.create();

entityManager.destroy(entity);
```
2. to test if entity valid:

```cpp
entityManager.isValid(entity);
```

3. to manage components:

```cpp
// ...
// to assign component to entity
entityManager.assign<Camera>(entity, near, far, fov, aspect);

//...
// to remove one or more components from entity
entityManager.remove<Camera, DirectionalLight>(entity);

// ...
// to get access to component of entity
entityManager.getComponent<Camera>(entity); // return pointer
// or to get access to several components of entity at once
entityManager.getComponents<Camera, Transform>(entity); // return tuple of pointers

// ...
// to test if entity has some component or components
entityManager.hasComponent<Camera>(entity); // return bool
entityManager.hasComponents<Camera, Transform>(entity); // return bitset

```

4. to get access to entities with necessary list of components:

 ```cpp
 auto view = entityManager.getView<Transform, Camera>();

 for (auto&& entity : view) {
    auto components = entityManager.getComponents<Transform, Camera>(entity);

    // do smth...
 }
 ```




