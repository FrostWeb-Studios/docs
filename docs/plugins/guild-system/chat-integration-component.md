---
title: UFWGuildChatIntegrationComponent
description: Optional component that bridges guild events with FWChatSystem for automated guild channel management.
---

# UFWGuildChatIntegrationComponent

**Header:** `FWGuildChatIntegrationComponent.h` | **Parent:** `UActorComponent`

Bridges guild lifecycle events with the FWChatSystem plugin to provide automatic guild chat channel management. When a guild is created, a dedicated chat channel is provisioned. When members join or leave, their channel access is updated accordingly.

!!! info "Optional Dependency"
    This component requires `FWChatSystem` to be enabled. If `FWChatSystem` is not present, the component logs a warning on initialization and disables itself. No guild functionality is lost -- only the automated chat channel features are unavailable.

---

## Class Declaration

```cpp
UCLASS(ClassGroup = (FrostWeb), meta = (BlueprintSpawnableComponent))
class FWGUILDSYSTEM_API UFWGuildChatIntegrationComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFWGuildChatIntegrationComponent();

    virtual void BeginPlay() override;
    virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;

    // --- Configuration ---
    UFUNCTION(BlueprintCallable, Category = "Guild|Chat")
    void SetChatChannelPrefix(const FString& Prefix);

    UFUNCTION(BlueprintPure, Category = "Guild|Chat")
    FString GetGuildChannelName() const;

    UFUNCTION(BlueprintPure, Category = "Guild|Chat")
    bool IsChatSystemAvailable() const;

    // --- Manual Controls ---
    UFUNCTION(BlueprintCallable, Category = "Guild|Chat")
    void SendGuildMessage(const FString& Message);

    UFUNCTION(BlueprintCallable, Category = "Guild|Chat")
    void SendGuildSystemMessage(const FString& Message);

    // --- Delegates ---
    UPROPERTY(BlueprintAssignable, Category = "Guild|Chat")
    FOnGuildChatMessageReceived OnGuildChatMessageReceived;

    UPROPERTY(BlueprintAssignable, Category = "Guild|Chat")
    FOnGuildChatChannelCreated OnGuildChatChannelCreated;

    UPROPERTY(BlueprintAssignable, Category = "Guild|Chat")
    FOnGuildChatChannelDestroyed OnGuildChatChannelDestroyed;
};
```

---

## Automated Behavior

The component listens to guild events from the sibling `UFWGuildManagerComponent` and `UFWGuildStateComponent` on the same actor, and performs the following actions automatically:

| Guild Event | Chat Action |
|---|---|
| Guild created | Creates a new chat channel named `{Prefix}_{GuildId}` |
| Guild disbanded | Destroys the guild chat channel |
| Member joined | Grants channel access to the new member |
| Member kicked/left | Revokes channel access |
| Member promoted/demoted | Updates channel role (moderator for ranks with `Kick` permission) |

---

## Functions

### SetChatChannelPrefix

```cpp
void SetChatChannelPrefix(const FString& Prefix);
```

Sets the prefix used when generating guild chat channel names. Default is `guild`.

| Parameter | Type | Description |
|---|---|---|
| `Prefix` | `FString` | Channel name prefix. The full channel name becomes `{Prefix}_{GuildId}`. |

---

### GetGuildChannelName

```cpp
FString GetGuildChannelName() const;
```

Returns the full channel name for the current guild. Returns an empty string if the player is not in a guild.

---

### IsChatSystemAvailable

```cpp
bool IsChatSystemAvailable() const;
```

Returns `true` if FWChatSystem is loaded and the integration is active. Use this to conditionally show guild chat UI elements.

---

### SendGuildMessage

```cpp
void SendGuildMessage(const FString& Message);
```

Sends a player message to the guild chat channel. The sender identity is automatically resolved from the owning player.

| Parameter | Type | Description |
|---|---|---|
| `Message` | `FString` | The message content to send. |

---

### SendGuildSystemMessage

```cpp
void SendGuildSystemMessage(const FString& Message);
```

Sends a system message to the guild chat channel. System messages are displayed differently in the chat UI (no sender name, distinct styling). Used internally to announce events like member joins and rank changes.

| Parameter | Type | Description |
|---|---|---|
| `Message` | `FString` | The system message content. |

---

## Delegates

### OnGuildChatMessageReceived

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnGuildChatMessageReceived,
    const FString&, SenderId,
    const FString&, Message);
```

Broadcast when a message is received on the guild chat channel. Use this to update a guild chat panel in your UI.

### OnGuildChatChannelCreated

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildChatChannelCreated,
    const FString&, ChannelName);
```

Broadcast when the guild chat channel is successfully created.

### OnGuildChatChannelDestroyed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildChatChannelDestroyed,
    const FString&, ChannelName);
```

Broadcast when the guild chat channel is destroyed (guild disbanded).

---

## Setup

### Adding to an Actor

The chat integration component should be added alongside the guild manager and state components on the same actor:

```cpp
// In your PlayerController or PlayerState constructor
GuildManager = CreateDefaultSubobject<UFWGuildManagerComponent>(TEXT("GuildManager"));
GuildState = CreateDefaultSubobject<UFWGuildStateComponent>(TEXT("GuildState"));
GuildChatIntegration = CreateDefaultSubobject<UFWGuildChatIntegrationComponent>(
    TEXT("GuildChatIntegration"));
```

### Conditional Compilation

If you want to completely exclude the chat integration from builds that do not include FWChatSystem:

```cpp
#if WITH_FWCHATSYSTEM
    GuildChatIntegration = CreateDefaultSubobject<UFWGuildChatIntegrationComponent>(
        TEXT("GuildChatIntegration"));
#endif
```

!!! note "Runtime vs Compile-Time"
    The component already handles runtime detection of FWChatSystem. Conditional compilation is only necessary if you want to eliminate the component's memory footprint entirely in builds without FWChatSystem.

---

## System Message Format

The following system messages are generated automatically:

| Event | Message Format |
|---|---|
| Member joined | `{PlayerName} has joined the guild.` |
| Member left | `{PlayerName} has left the guild.` |
| Member kicked | `{PlayerName} was removed from the guild.` |
| Member promoted | `{PlayerName} has been promoted to {RankName}.` |
| Member demoted | `{PlayerName} has been demoted to {RankName}.` |
| Rank edited | `Guild ranks have been updated.` |
| Guild info edited | `Guild information has been updated.` |

---

## Usage Example

=== "C++"

    ```cpp
    void UGuildChatWidget::Initialize()
    {
        auto* ChatIntegration =
            GetOwningPlayerController()
                ->FindComponentByClass<UFWGuildChatIntegrationComponent>();

        if (ChatIntegration && ChatIntegration->IsChatSystemAvailable())
        {
            ChatIntegration->OnGuildChatMessageReceived.AddDynamic(
                this, &UGuildChatWidget::OnMessageReceived);
            SetVisibility(ESlateVisibility::Visible);
        }
        else
        {
            SetVisibility(ESlateVisibility::Collapsed);
        }
    }

    void UGuildChatWidget::OnSendButtonClicked()
    {
        auto* ChatIntegration =
            GetOwningPlayerController()
                ->FindComponentByClass<UFWGuildChatIntegrationComponent>();

        if (ChatIntegration)
        {
            ChatIntegration->SendGuildMessage(MessageInputBox->GetText().ToString());
            MessageInputBox->SetText(FText::GetEmpty());
        }
    }
    ```

=== "Blueprint"

    1. Add **FW Guild Chat Integration Component** to the same actor as your guild components.
    2. On your chat widget, call **Is Chat System Available** to decide whether to show the guild chat tab.
    3. Bind **On Guild Chat Message Received** to append messages to your chat log.
    4. Wire a send button to call **Send Guild Message** with the text input value.
