# (Photon) Fusion 2 Shared Ders Notları

- Projemizi oluşturduktan sonra Hierarchy üzerine sağ tıklayıp **Fusion > Scene > Setup Networkin in the Scene** seçeneğini seçiyoruz.
- Prototype Network Start ve Prototype Runner adında 2 adet objeyi bizim için oluşturacak.
- Player Spawner kodumuzu yazıp Prototype Runner objesine ekleyeceğiz.

## Player Spawner

```c#
using Fusion;
using UnityEngine;

public class PlayerSpawner : SimulationBehaviour, IPlayerJoined
{
    [SerializeField] private GameObject playerCharacter; //Player için hazırladığımız GameObject
    public void PlayerJoined(PlayerRef player)
    {
        Runner.Spawn(playerCharacter, new Vector3(0, 1f, 0), Quaternion.identity);
    }
}
```
- **Player Prefabımıza Network Object, Network Transform, Character Controller eklediğimizden emin olmalıyız.**
- Oyunumuzu Shared modunda çalıştırdığımızda sunucunun bizim belirlediğimiz Prefabı Spawn ettiğini görebiliriz.

## Player Movement

- Karakterimizin hareket ve zıplamasını sağlayan kodumuzu yazıyoruz.

```c#
using Fusion;
using UnityEngine;

public class PlayerMovement : NetworkBehaviour
{
    CharacterController characterController;
    Vector3 velocity;
    bool isJumpPressed;
    public float gravity = -9.81f;
    public float playerSpeed = 5f;
    public float jumpForce = 5f;
    private void Start()
    {
        characterController = GetComponent<CharacterController>();
    }
    private void Update()
    {
        if(Input.GetButtonDown("Jump")) { isJumpPressed = true; }
    }
    public override void FixedUpdateNetwork()
    {
        if (!HasStateAuthority) { return; } //Bu satır istemcinin bize ait olup olmadığını kontrol eden önemli bir parçadır.
        if (characterController.isGrounded) { velocity = new Vector3(0, -1, 0); }
        Vector3 move = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical")) * Runner.DeltaTime * playerSpeed;
        velocity.y += gravity * Runner.DeltaTime;
        if (isJumpPressed && characterController.isGrounded) { velocity.y += jumpForce; }
        characterController.Move(move + velocity * Runner.DeltaTime);
        if (move != Vector3.zero) { gameObject.transform.forward = move; }
        isJumpPressed = false;
    }
}
```
- Sunucu ile senkron davranması için **FixedUpdateNetwork()** metodunu ve **Runner.DeltaTime** zamanlayıcısını kullanıyoruz.
