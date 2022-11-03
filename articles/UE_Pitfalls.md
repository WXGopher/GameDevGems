### Pitfalls in UE4

Here is a collection of random pitfalls (~~they are feature not bugs~~) I've encountered. 

//TOC here

#### On DDC cache /UE4.27

UE uses DDC to cache cooked data and facilitates user operations. 

#### ApplyWorldOffset /UE4.27

If we have several levels with offset in a game, after unloading and reloading levels you may find some level components not set up properly.

```c++
void USceneComponent::ApplyWorldOffset(const FVector& InOffset, bool bWorldShift)
{
	Super::ApplyWorldOffset(InOffset, bWorldShift);
	
	// Calculate current ComponentToWorld transform
	// We do this because at level load/duplication ComponentToWorld is uninitialized
	{
		ComponentToWorld = CalcNewComponentToWorld(GetRelativeTransform());
	}

	// Update bounds
	Bounds.Origin+= InOffset;

	// Update component location
	if (GetAttachParent() == nullptr || IsUsingAbsoluteLocation())
	{
		SetRelativeLocation_Direct(GetComponentLocation() + InOffset);
		
		// Calculate the new ComponentToWorld transform
		ComponentToWorld = CalcNewComponentToWorld(GetRelativeTransform());
	}

	// Physics move is skipped if physics state is not created or physics scene supports origin shifting
	// We still need to send transform to physics scene to "transform back" actors which should ignore origin shifting
	// (such actors receive Zero offset)
	const bool bSkipPhysicsTransform = (!bPhysicsStateCreated || (bWorldShift && FPhysScene::SupportsOriginShifting() && !InOffset.IsZero()));
	OnUpdateTransform(SkipPhysicsToEnum(bSkipPhysicsTransform));
	
	// We still need to send transform to RT to "transform back" primitives which should ignore origin shifting
	// (such primitives receive Zero offset)
	if (!bWorldShift || InOffset.IsZero())
	{
		MarkRenderTransformDirty();
	}

	// Update physics volume if desired	
	if (bShouldUpdatePhysicsVolume && !bWorldShift)
	{
		UpdatePhysicsVolume(true);
	}

	// Update children
	for (USceneComponent* ChildComp : GetAttachChildren())
	{
		if(ChildComp != nullptr)
		{
			ChildComp->ApplyWorldOffset(InOffset, bWorldShift);
		}
	}
}
```

How unload works is the actor to be unloaded traversing through inheritance tree and executing `ApplyWorldOverset ` (as seen in `	// Update children` above). This happens recursively, and all children components may have spatial data offset by `InOffset` (in their `ApplyWorldOffset`).

This causes problem on load. The root component still calls `ApplyWorldOffset` on children components; however, by the time this gets called, children components are not initialized yet (`ChildComp` is `nullptr`), hence spatial data offset by `InOffset` in unloading process may not get compensated, resulting weirdly shifted components.

How UE internally solved this problem is in each components' `OnRegister`, spatially offset data are recalculated. Take `UCableComponent` for example:

```c++
void UCableComponent::OnRegister()
{
	Super::OnRegister();

	const int32 NumParticles = NumSegments+1;

	Particles.Reset();
	Particles.AddUninitialized(NumParticles);

	FVector CableStart, CableEnd;
	GetEndPositions(CableStart, CableEnd);

	const FVector Delta = CableEnd - CableStart;

	for(int32 ParticleIdx=0; ParticleIdx<NumParticles; ParticleIdx++)
	{
		FCableParticle& Particle = Particles[ParticleIdx];

		const float Alpha = (float)ParticleIdx/(float)NumSegments;
		const FVector InitialPosition = CableStart + (Alpha * Delta);

		Particle.Position = InitialPosition;
		Particle.OldPosition = InitialPosition;
		Particle.bFree = true; // default to free, will be fixed if desired in TickComponent
	}
}
```

As shown above, cable data are recalculated using `CableStart` and `CableEnd`, which are recalculated and do not depend on last saved status.