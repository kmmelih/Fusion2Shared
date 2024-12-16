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
    [SerializeField] private GameObject playerCharacter;
    public void PlayerJoined(PlayerRef player)
    {
        if (Runner.LocalPlayer == player)
        {
            Runner.Spawn(playerCharacter, new Vector3(0, 1, 0), Quaternion.identity, player);
        }
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
    public Camera camera;
    private void Awake()
    {
        characterController = GetComponent<CharacterController>();
    }
    private void Update()
    {
        if(Input.GetButtonDown("Jump")) { isJumpPressed = true; }
    }
    public override void FixedUpdateNetwork()
    {
        if (!HasStateAuthority) { return; }
        if (characterController.isGrounded) { velocity = new Vector3(0, -1, 0); }
        Quaternion cameraRotationY = Quaternion.Euler(0,camera.transform.rotation.eulerAngles.y,0);
        Vector3 move = cameraRotationY * new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical")) * Runner.DeltaTime * playerSpeed;
        velocity.y += gravity * Runner.DeltaTime;
        if (isJumpPressed && characterController.isGrounded) { velocity.y += jumpForce; }
        characterController.Move(move + velocity * Runner.DeltaTime);
        if (move != Vector3.zero) { gameObject.transform.forward = move; }
        isJumpPressed = false;
    }

    public override void Spawned()
    {
        if (HasStateAuthority) 
        {
            camera = Camera.main;
            camera.GetComponent <FirstPersonCamera>().Target = transform;
        }
    }
}
```
- Sunucu ile senkron davranması için **FixedUpdateNetwork()** metodunu ve **Runner.DeltaTime** zamanlayıcısını kullanıyoruz.

## First Person Camera

- Karakterimizi takip eden First Person Camera kodlarını yazıyoruz ve Main Camera'ya ekliyoruz.

```c#
using Fusion;
using UnityEngine;

public class FirstPersonCamera : MonoBehaviour
{
    public Transform Target;
    public float MouseSens = 12f;

    float mouseX, mouseY;
    float verticalRotation, horizontalRotation;

    private void LateUpdate()
    {
        if(Target == null) {  return; }
        transform.position = Target.position;
        mouseX = Input.GetAxis("Mouse X");
        mouseY = Input.GetAxis("Mouse Y");
        verticalRotation -= mouseY * MouseSens;
        verticalRotation = Mathf.Clamp(verticalRotation, -70f, 70f);
        horizontalRotation += mouseX * MouseSens;
        transform.rotation = Quaternion.Euler(verticalRotation, horizontalRotation, 0f);
    }
}
```
