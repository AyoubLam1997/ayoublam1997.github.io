---
title: MySQL Database
date: 2024-07-07 12:00:00 -500
categories: [project,study,solo]
tags: [mysql, csharp, php, unity]     # TAG names should always be lowercase
---

# MySQL Database

For a project when I was at Grafisch Lyceum Utrecht, I created a MySQL database. A user can register an account that holds the user's email, password & data depending on the game. In this case, in this space shooter, the user can upgrade their ship. The upgrade increases the ship's damage, health & speed.

I run the server on EasyPHP Devserver 17.0 & handle MySQL on phpMyAdmin.

The system makes use of a Token-based authentication where when a player is logging in, they get a randomly generated access token. Whilst the token is active, the user can navigate through the game & get access to their data such as the ship health & damage.

First, if the player has an account, the player attempts to login in the game. After the player has put in their credentials, the game tries to check if the credentials are correct & if so, let's the player is then logged in.

```cs
[System.Serializable]
public class LoginResponse
{
    public string Token;
}

public class Login : Monobehaviour
{
    private TMPro.TextMeshProUGUI m_Text;

    [SerializeField]
    private InputField m_UserNameInput, m_PasswordInput;

    public void CheckAccount()
    {
        StartCoroutine(LoginAccount());
    }

    private IEnumerator LoginAccount()
    {
        // Create WWWForm to generate form data
        WWWForm form = new WWWForm();

        // Add data to form
        form.addField("name", m_UserNameInput.text);
        form.addField("password", m_PasswordInput.text);

        // Create WWW & get the login URL
        WWW www = new WWW("http://127.0.0.1/edsa-Example/login.php");

        yield return www;

        // Login failed because either login credentials are incorrect or server is down.
        if(form.error != null)
        {
            Debug.Log("Login failed.");
        }
        else
        {
            // Receive login data & ship data
            LoginResponse response = JsonUtility.FromJson<LoginResponse>(www.text);
            BaseResponse baseData = JsonUtility.FromJson<BaseResponse>(www.text);

            UserData.UserName = m_UserNameInput.text;
            UserData.Score = baseData.Score;
            UserData.Health = baseData.Health;
            UserData.Damage = baseData.Damage;

            SceneManager.LoadScene("Upgrade");
        }
    }
}
```

On the server side, it receives the login credentials the player has filled in & checks if the data is correct if the player already has an account and/or if the account even exists in the first place.

```php
<?php
require "ConnectUnity.php"
require "PlayerData.php"

$username = $_POST["name"];
$password = $_POST["password"];
$hash = password_hash($password, PASSWORD_BCRYPT);

$sql = "SELECT * FROM accounts WHERE username = '{$username}'";

// Check if the username exist in the SQL server
$result = $connect->query($sql);

if($result->rows_num > 0)
{
    $row = $result->fetch_assoc();
    
    // Check if the password of the account is correct
    if(password_verify($password, $row['hash']))
    {
        // Generate a unique token for the player to use to navigate the game
        $token = RandomString(128);
        
        $sql = "UPDATE accounts SET token '{$token}' WHERE id = {$row, ['id']}";

        // creating & setting the token is successful
        if($connect->query($sql) === true)
        {
            // Grab & set the user data
            $response = new LoginResponse();
            $response->token = $token;

            $score = $row['score'];
            $health = $row['health'];
            $damage = $row['damage'];

            $response = new BaseResponse();
            $response->score = $score;
            $response->health = $health;
            $response->damage = $damage;

            // Send the data to the user in the form of a json string
            echo json_encode($response);
        }
        else
        {
            $connect->error;
        }
    }
    else
    {
        echo "Password is incorrect";
    }
}
else
{
    echo "Account doesn't exist";
}

?>
```

Below is a video of me demonstrating the project.

[![MySQL Space-Shooter Project](https://img.youtube.com/vi/zpE9lKieaoU&t=2s/0.jpg)](https://www.youtube.com/watch?v=zpE9lKieaoU&t=2s "MySQL, login, save player data & space shooter")
