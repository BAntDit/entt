
# Enttx

Compile-time and header-only [Entity-Component-System](https://en.wikipedia.org/wiki/Entity_component_system) library.

## Overview

There is a lot of ECS implementations you can find in the github, at least i've seen many of them,
but didn't find something suitable for my goals. Some of them do a lot of job in runtime,
some of then are complicated for integration, some of them have tons of unecessary for me functionality and etc.
Finally i decided to create another one.
I don't pretend for best solution i just created another one solution suitable for my goals
and shared it here if somebody finds my solutions usefull for his own projects as well.

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

Entity class is just wrapping uint64_t numbers to present entity version
and entity index to associate entity with set of components.

### components

Components stores data related to objects and have as little logic as possible (typically POD types).
Components must be default constructible.

### storages

Enttx provides default storage type for components and CRTP template to create custom storage if necessary.
Storages must be default constuctible.

### entities and components management

EntityManager class provides all necessary functionality to manage entities and those classes

1. to create / destroy entity

```cpp
auto entity = entityManager.create();

entityManager.destroy(entity);
```

// or to create many at once:

```cpp
auto entities = entityManager.createMany(std::array<enttx::Entity, 1000>());
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

 for (auto&& [entity, transform, camera] : view) {
    // do smth...
 }
 ```

 // or to get all entities that has at least one component

  ```cpp
  auto view = entityManager.getView<>();

  for (auto&& [entity] : view) {
      // do smth...
  }
  ```

 // or to get direct access to component's storage

 ```cpp
 auto storage = entityManager.getStorage<Camera>(); // storage of camera's components
 ```

5. to apply changes for any components of entity:

```cpp
auto fn = [](auto&& component) -> void {
    if constexpr (std::is_same_v<std::decay_t<decltype(component)>, Camera>) {
        // do this...
    }

    if constexpr (std::is_same_v<std::decay_t<decltype(component)>, DirectionalLight>) {
        // do that...
    }
};

entityManager.applyForComponents<Camera, DirectionalLight>(entity, fn);
```

### systems

Enttx provides BaseSystem CRTP pattern to create new system.

```cpp
class KinematicsSystem: public enttx::BaseSystem<KinematicsSystem>
{
public:
    // to let the system manager know type of components system is going to work with
    using tag_t = mp::type_list<Acceleration, Velocity>;

    void init() { }

    template<typename EntityManager, size_t STAGE>
    void update(EntityManager& entityManager)
    {
        if constexpr (STAGE == STAGES::EARLY_UPDATE) {
            auto view = entityManager.template getView<Velocity, Acceleration>();

            for (auto&& [entity, velocity, acceleration] : view) {
                // do smth...
            }
        }

        if constexpr (STAGE == STAGES::LATE_UPDATE) {
            // do smth else...
        }
    }
};
```

### systems management

SystemManager calls update sequentially stage by stage once a frame, system by system once a stage.

1. to start updating for all systems, just call:
```cpp
systemManager.update();
```
2. to test if system manager has system that works with defined list of components:

```cpp
if constexpr (SystemManager::template has_system_for_components_v<Velocity>) {
    // do smth...
}
```

3. to get the tuple of systems that works with defined list of components:
```cpp
auto systems = systemManager.getSystemsForComponents<Acceleration, Velocity>();
```

## Dependencies

* [easy-mp](https://github.com/BAntDit/easy-mp) - meta-programming library


