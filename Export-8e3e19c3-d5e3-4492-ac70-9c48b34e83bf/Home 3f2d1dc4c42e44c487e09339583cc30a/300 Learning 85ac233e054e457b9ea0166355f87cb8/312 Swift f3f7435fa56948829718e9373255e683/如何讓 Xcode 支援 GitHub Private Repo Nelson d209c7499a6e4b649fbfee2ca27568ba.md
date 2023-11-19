# 如何讓 Xcode 支援 GitHub Private Repo | Nelson

我們的 codebase 有用到放在 GitHub private repo 的 Swift package，為了讓 Xcode 能夠存取 private repo，必須讓 Xcode 認得 SSH key 才行。

因為 Xcode 只支援 RSA / DSA / ECDSA 這三種演算法的 SSH key，但 [GitHub 不支援 RSA 跟 DSA](https://github.blog/2021-09-01-improving-git-protocol-security-github/) 了，所以我們只剩下 **ECDSA** 可以用。

*如果當時按照 GitHub 的文件建立 SSH key，它很有可能是 ED25519，這個演算法 Xcode 認不得，所以我們要新增一個新的 SSH key。*

## 修改 SSH Key

新增 SSH key 的方法可以參考 [GitHub 的文件](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)，唯二要修改的地方是

1. 把 `ssh-keygen -t ed25519 -C "your_email@example.com"` 改成 `ssh-keygen -t ecdsa -b 521 -C "your_email@example.com"`
2. 把 `id_ed25519` 改成 `id_ecdsa`

然後參考另一份 [GitHub 文件](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)，把 SSH key 加到你的 GitHub account。

## Xcode 新增 GitHub account

從 Xcode `Preference -> Accounts` 新增一個 GitHub account，

- Account 就是登入 GitHub 的信箱
- Token 從 [GitHub Settings](https://github.com/settings/tokens) 建立，記得 Xcode 需要的權限都要打勾，然後設為永不過期

![%E5%A6%82%E4%BD%95%E8%AE%93%20Xcode%20%E6%94%AF%E6%8F%B4%20GitHub%20Private%20Repo%20Nelson%20d209c7499a6e4b649fbfee2ca27568ba/1651241495786.png](%E5%A6%82%E4%BD%95%E8%AE%93%20Xcode%20%E6%94%AF%E6%8F%B4%20GitHub%20Private%20Repo%20Nelson%20d209c7499a6e4b649fbfee2ca27568ba/1651241495786.png)

建立帳號之後，記得把 SSH Key 設為剛剛建立的 ECDSA，這樣就完成啦。

![%E5%A6%82%E4%BD%95%E8%AE%93%20Xcode%20%E6%94%AF%E6%8F%B4%20GitHub%20Private%20Repo%20Nelson%20d209c7499a6e4b649fbfee2ca27568ba/1651241676938.png](%E5%A6%82%E4%BD%95%E8%AE%93%20Xcode%20%E6%94%AF%E6%8F%B4%20GitHub%20Private%20Repo%20Nelson%20d209c7499a6e4b649fbfee2ca27568ba/1651241676938.png)