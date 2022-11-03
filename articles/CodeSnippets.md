### Code Snippets



Disclaimer: I am far from a UE expert.  Please take my words with a grain of salt.

#### Create blueprint in C++ /4.26

```c++
const FString AssetPath = TEXT("Game/Artwork/MyAsset");

UClass* MyClassType = MyClass::StaticClass();
UPackage* MyPackage = CreatePackage(*AssetPath);

UClass* BlueprintClass = nullptr;
UClass* BlueprintGeneratedClass = nullptr;

IKismetCompilerInterface& KismetCompilerModule = FModuleManager::LoadModuleChecked<IKismetCompilerInterface>("KismetCompiler");
KismetCompilerModule.GetBlueprintTypesForClass(MyClassType, BlueprintClass, BlueprintGeneratedClass);

UBlueprint* MyBlueprint = FKismetEditorUtilities::CreateBlueprint(MyClassType, MyPackage, FName(TEXT("BlueprintName")), EBlueprintType::BPTYPE_Normal, BlueprintClass, BlueprintGeneratedClass);

FAssetRegistryModule::AssetCreated(MyBlueprint);

MyBlueprint->MarkPackageDirty();
MyPackage->SetDirtyFlag(true);
```

How I came this up: single-stepped how UE creates a blueprint using "create blueprint class".

If you are going to automatically generate a bunch of blueprints, I would highly against creating blueprints on your own. Instead, just duplicate a bunch of blueprint templates and make modifications to them. 



#### Set properties in blueprint /4.26

It really depends the scenario. Basically the obstacles come from blueprints are not "prefabs" (or a collection of data). People normally use blueprint as using prefabs in Unity, but in fact blueprints are classes, meaning lots of entities (especially components) are not instantiated until you pull up a blueprint editor. 

Blueprints are not meant to "save data", although people use them to. Lots of information come from their `Simple Construction Script` (something like a ctor in a normal C++ class). 

I'm talking about "setting up properties" not just "reading properties", so things like instantiating an actor blueprint as a world Actor and make modifications to it is not an option. 

1. If you have a blueprint *directly inherited* from a C++ class,, you can do this to retrieve its components

    ```C++
    const TArray<USCS_Node*>& BPNodes = BPPtr->SimpleConstructionScript->GetAllNodes();
    
    for(USCS_Node* SCSNode : BPNodes)
    {
        if (SCSNode->GetVariableName() == FName(TEXT("MeshName")))
        {
    	// cast it to whatever you think it should be
        UStaticMeshComponent* SMC = Cast<UStaticMeshComponent>(SCSNode->ComponentTemplate);
    	// Do whatever you want...
         }
    }
    ```

2. For blueprint *variables*,  you can also use reflection:

    ```c++
    // for example, we want to set a static mesh SomeNameYouWant
    const FName CompName = FName(TEXT("SomeNameYouWant"));
    FProperty* Prop = BPPtr->GeneratedClass->FindPropertyByName(CompName);
    FObjectProperty* ObjProp = CastField<FObjectProperty>(Prop);
    if (ObjProp)
    {
    	void* ValPtr = ObjProp->ContainerPtrToValuePtr<void>(BPPtr->GeneratedClass->GetDefaultObject());
    	ObjProp->SetPropertyValue(ValPtr, MeshToSet); // if you know you are setting up a static mesh
    }
    
    ```

3. However, if you are dealing with a blueprint *inherited from* a *blueprint*, `SimpleConstructionScript` will return empty nodes, but you can do this:

    ```C++
    
    TArray<USCS_Node*> AllNodes;
    for (auto Dep : BPPtr->CachedDependencies)
    {
        TArray<USCS_Node*> Nodes = Dep->SimpleConstructionScript->GetAllNodes();
        AllNodes.Append(Nodes);
    }
    // harvest all components into an array so we can set them
    TArray<UActorComponent*> HarvestComponents;
    for (auto Node : AllNodes)
    {
        UActorComponent* OverriddenComponent = nullptr;
    
        FComponentKey Key(Node);
    
        const bool bBlueprintCanOverrideComponentFromKey = Key.IsValid()
                  && BPPtr
                  && BPPtr->ParentClass
                  && BPPtr->ParentClass->IsChildOf(Key.GetComponentOwner());
    
        if (bBlueprintCanOverrideComponentFromKey)
        {
            const bool bCreateIfNecessary = true;
            UInheritableComponentHandler* InheritableComponentHandler = BPPtr->GetInheritableComponentHandler(bCreateIfNecessary);
            if (InheritableComponentHandler)
            {
                OverriddenComponent = InheritableComponentHandler->GetOverridenComponentTemplate(Key);
                if (!OverriddenComponent &amp;&amp; bCreateIfNecessary)
                {
                    OverriddenComponent = InheritableComponentHandler->CreateOverridenComponentTemplate(Key);
                }
            }
            TArray<UActorComponent*> OutArray;
            InheritableComponentHandler->GetAllTemplates(OutArray, true);
            for (int32 i = 0; i < OutArray.Num(); ++i) {
                HarvestComponents.AddUnique(OutArray[i]);
            }
        }
    }
    
    // you can use Component->GetName() to query the specific component you want. Howver, note that the name is not what it is displayed in the blueprint editor. Put a breakpoint there to see what it looks like.
    ```


#### Dump PX profiling information

PhysX has its own cycle counter, tagged with `PX_PROFILE_ZONE`. These information by default will not be collected in UE. Here is a diff patch to tell UE to collect information from PhysX.

```C++
diff --git a/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysLevel.cpp b/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysLevel.cpp

--- a/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysLevel.cpp
+++ b/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysLevel.cpp
@@ -287,6 +287,10 @@ bool InitGamePhys()
 		DeferredPhysResourceCleanup();
 	});
 	
+#if PHYSICS_INTERFACE_PHYSX && STATS
+	physx::PxProfilerCallback* profiler = new FPhysXProfilerCallback();
+	PxSetProfilerCallback(profiler);
+#endif
 
 	// Message to the log that physics is initialised and which interface we are using.
 	UE_LOG(LogInit, Log, TEXT("Physics initialised using underlying interface: %s"), *FPhysicsInterface::GetInterfaceDescription());
@@ -309,6 +313,17 @@ void TermGamePhys()
 		FPhysxSharedData::Terminate();	//early out before TermGamePhysCore so kill this - not sure if this is a real case we even care about
 		return;
 	}
+
+#if STATS
+	FPhysXProfilerCallback* profiler = static_cast<FPhysXProfilerCallback*>(PxGetProfilerCallback());
+
+	if (profiler != nullptr)
+	{
+		delete profiler;
+	}
+	PxSetProfilerCallback(0);
+#endif
+
 #endif
 
 	if (GPhysCommandHandler != NULL)
diff --git a/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysXSupport.cpp b/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysXSupport.cpp

--- a/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysXSupport.cpp
+++ b/Engine/Source/Runtime/Engine/Private/PhysicsEngine/PhysXSupport.cpp
@@ -624,20 +624,21 @@ PxCollection* MakePhysXCollection(const TArray<UPhysicalMaterial*>& PhysicalMate
 
 void* FPhysXProfilerCallback::zoneStart(const char* eventName, bool detached, uint64_t contextId)
 {
-	if(GCycleStatsShouldEmitNamedEvents > 0)
+#if !(UE_BUILD_TEST || UE_BUILD_SHIPPING)
 	{
 		FPlatformMisc::BeginNamedEvent(FColor::Red, *FString::Printf(TEXT("PHYSX: %s"), StringCast<TCHAR>(eventName).Get()));
 	}
-
+#endif
 	return nullptr;
 }
 
 void FPhysXProfilerCallback::zoneEnd(void* profilerData, const char* eventName, bool detached, uint64_t contextId)
 {
-	if(GCycleStatsShouldEmitNamedEvents > 0)
+#if !(UE_BUILD_TEST || UE_BUILD_SHIPPING)
 	{
 		FPlatformMisc::EndNamedEvent();
 	}
+#endif
 }
 
 void FPhysXMbpBroadphaseCallback::onObjectOutOfBounds(PxShape& InShape, PxActor& InActor)
```