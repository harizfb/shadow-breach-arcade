CameraRail.cs
using UnityEngine;

public class CameraRail : MonoBehaviour {
    public Transform[] nodes;
    public float speed = 3f;
    public float pauseDuration = 1f;
    int index = 0;
    float pauseTimer;

    void Update() {
        if (index >= nodes.Length) return;

        if (Vector3.Distance(transform.position, nodes[index].position) > 0.1f) {
            transform.position = Vector3.MoveTowards(transform.position, nodes[index].position, speed * Time.deltaTime);
            transform.rotation = Quaternion.Lerp(transform.rotation, nodes[index].rotation, Time.deltaTime * 2f);
        } else {
            pauseTimer += Time.deltaTime;
            if (pauseTimer >= pauseDuration) {
                index++;
                pauseTimer = 0;
            }
        }
    }
}

PlayerShooter.cs
using UnityEngine;
using System.Collections;

public class PlayerShooter : MonoBehaviour {
    public GameObject bulletPrefab;
    public Transform firePoint;
    public int ammo = 6;
    public float reloadTime = 2f;
    public AudioSource fireSound, reloadSound;

    bool isReloading;

    void Update() {
        if (isReloading) return;

        if (Input.GetButtonDown("Fire1") && ammo > 0) {
            Shoot();
        }

        if (Input.GetKeyDown(KeyCode.R)) {
            StartCoroutine(Reload());
        }
    }

    void Shoot() {
        Instantiate(bulletPrefab, firePoint.position, firePoint.rotation);
        fireSound?.Play();
        ammo--;
    }

    IEnumerator Reload() {
        isReloading = true;
        reloadSound?.Play();
        yield return new WaitForSeconds(reloadTime);
        ammo = 6;
        isReloading = false;
    }
}

Bullet.cs
using UnityEngine;

public class Bullet : MonoBehaviour {
    public float speed = 50f;

    void Update() {
        transform.Translate(Vector3.forward * speed * Time.deltaTime);
    }

    void OnTriggerEnter(Collider other) {
        if (other.CompareTag("Enemy")) {
            other.GetComponent<Enemy>()?.TakeDamage(50);
        }
        Destroy(gameObject);
    }
}

Enemy.cs
using UnityEngine;

public class Enemy : MonoBehaviour {
    public int health = 100;
    public GameObject deathEffect;

    public void TakeDamage(int amount) {
        health -= amount;
        if (health <= 0) Die();
    }

    void Die() {
        if (deathEffect) Instantiate(deathEffect, transform.position, Quaternion.identity);
        Destroy(gameObject);
        ScoreManager.Instance?.AddScore(100);
    }
}

ScoreManager.cs
using UnityEngine;
using UnityEngine.UI;

public class ScoreManager : MonoBehaviour {
    public static ScoreManager Instance;
    public Text scoreText;
    int score;

    void Awake() => Instance = this;

    public void AddScore(int value) {
        score += value;
        UpdateUI();
    }

    void UpdateUI() {
        if (scoreText)
            scoreText.text = "Score: " + score.ToString("D5");
    }
}

