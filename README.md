resource "aws_iam_user" "s3_media_user" {
  name = "s3-media-uploader"
}

resource "aws_iam_access_key" "s3_media_key" {
  user = aws_iam_user.s3_media_user.name
}

resource "aws_iam_policy" "s3_media_policy" {
  name        = "S3MediaUploadPolicy"
  description = "Allows uploading media to S3 only"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = [
          "s3:PutObject"
        ]
        Resource = "arn:aws:s3:::afnan/*"
      }
    ]
  })
}

resource "aws_iam_user_policy_attachment" "attach_policy" {
  user       = aws_iam_user.s3_media_user.name
  policy_arn = aws_iam_policy.s3_media_policy.arn
}







resource "aws_security_group" "mern_app_sg" {
  name        = "mern-app-security-group"
  description = "Security group for MERN App"

  vpc_id = "vpc-0a1b2c3d4e5f67890" 

  ingress = [
    {
      description = "Allow SSH access"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]     },
    {
      description = "Allow HTTP access"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      description = "Allow App Port"
      from_port   = 5000
      to_port     = 5000
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]

  egress = [
    {
      description = "Allow all outbound traffic"
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}



resource "aws_s3_bucket" "frontend_bucket" {
  bucket = "your-frontend-bucket-name"
}

resource "aws_s3_bucket_website_configuration" "frontend_website" {
  bucket = aws_s3_bucket.frontend_bucket.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }
}

resource "aws_s3_bucket_policy" "frontend_policy" {
  bucket = aws_s3_bucket.frontend_bucket.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource  = "arn:aws:s3:::afnann/*"
      }
    ]
  })
}





resource "aws_s3_bucket" "media_bucket" {
  bucket = "your-media-bucket-name"
}

resource "aws_s3_bucket_policy" "media_policy" {
  bucket = aws_s3_bucket.media_bucket.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = { "AWS" = "arn:aws:iam::afnan290/s3-media-uploader" }
        Action    = ["s3:PutObject"]
        Resource  = "arn:aws:s3:::afnannn/*"
      }
    ]
  })
}

resource "aws_s3_bucket_cors_configuration" "media_cors" {
  bucket = aws_s3_bucket.media_bucket.id

  cors_rule {
    allowed_methods = ["PUT", "POST"]
    allowed_origins = ["*"]
    allowed_headers = ["Authorization"]
    max_age_seconds = 3000
  }
}




resource "aws_launch_template" "mern_backend_template" {
  name_prefix   = "mern-backend"
     = "ami-1234567890abcdef0"
  instance_type = "t3.micro"
  key_name      = "afnaf"

  network_interfaces {
    associate_public_ip_address = true
    security_groups             = [aws_security_group.mern_app_sg.id]
  }

  user_data = base64encode(<<EOF
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1
apt update -y
apt install -y git curl unzip tar gcc g++ make
su - ubuntu -c 'curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash'
su - ubuntu -c 'export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh" && nvm install --lts'
su - ubuntu -c 'export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh" && npm install -g pm2'
su - ubuntu -c 'mkdir -p ~/logs'
EOF
  )
}
![photo_5843656911969438807_y](https://github.com/user-attachments/assets/8c44ea39-a0d6-4f82-840b-f56e48ec67a3)

